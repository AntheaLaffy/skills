# 脚本骨架：状态计算器 + 校验

一个文件搞定三件事：解析事实源 → 生成派生块 → 校验。约 80—150 行，标准库即可。
下面是通用骨架，把 `load_*` 换成你的项目结构。

```python
#!/usr/bin/env python3
"""status — 全书/全站状态的唯一计算器。"""
import argparse, re, sys
from dataclasses import dataclass, field
from pathlib import Path

BEGIN = "<!-- BEGIN generated:{name} -->"
END = "<!-- END generated:{name} -->"


@dataclass
class Item:            # 一个章/一篇文章/一个里程碑
    num: int
    title: str
    group: int
    written: bool = False
    size: int = 0


@dataclass
class Group:           # 卷/里程碑阶段
    num: int
    name: str
    range: tuple[int, int] | None
    items: list[Item] = field(default_factory=list)


# ---- 事实源 1：计划清单（标题行） -----------------------------------------
PLAN_RE = re.compile(r"^##\s*第(\d+)章《(.+?)》")
HEAD_RE = re.compile(r"^#\s*第(\d+)卷《(.+?)》（第(\d+)[—\-](\d+)章")


def load_plan(root: Path) -> list[Group]:
    groups = []
    for path in sorted((root / "大纲/卷纲").glob("*.md")):
        text = path.read_text(encoding="utf-8")
        head = HEAD_RE.search(text)
        group = Group(int(head.group(3)) and 0 or 0, head.group(2),
                      (int(head.group(3)), int(head.group(4))))
        for line in text.splitlines():
            m = PLAN_RE.match(line)
            if m:
                group.items.append(Item(int(m.group(1)), m.group(2), group.num))
        groups.append(group)
    return groups


# ---- 事实源 2：产物（文件存在即已写，首行是标题） -------------------------
def load_artifacts(groups: list[Group], body: Path) -> list[str]:
    problems = []
    by_num = {i.num: i for g in groups for i in g.items}
    for path in sorted(body.glob("*/*.md")):
        m = re.match(r"^第(\d+)章-(.+)\.md$", path.name)
        if not m:
            problems.append(f"文件名不合规：{path}")
            continue
        num = int(m.group(1))
        first = path.read_text(encoding="utf-8").splitlines()[0]
        title = re.match(r"^#\s*第\d+章\s*(.+?)\s*$", first)
        item = by_num.get(num)
        if item is None:
            problems.append(f"第{num}章有正文但计划里没有")
            continue
        if title and title.group(1) != item.title:
            problems.append(f"第{num}章标题不一致：计划《{item.title}》／正文《{title.group(1)}》")
        item.written = True
        item.size = len(path.read_text(encoding="utf-8"))
    return problems


# ---- 派生文本 -------------------------------------------------------------
def block_status(groups: list[Group]) -> str:
    done = [i for g in groups for i in g.items if i.written]
    nxt = next((i for g in groups for i in g.items if not i.written), None)
    lines = [f"- 已写 {len(done)} / {sum(len(g.items) for g in groups)}"]
    if nxt:
        lines.append(f"- 下一章：{nxt.num}《{nxt.title}》")
    return "\n".join(lines)


# ---- 生成块读写 -----------------------------------------------------------
TARGETS = {"AGENTS.md": "status"}          # 文件 -> 生成块名


def pattern(name: str) -> re.Pattern[str]:
    return re.compile(re.escape(BEGIN.format(name=name)) + r"\n(.*?)"
                      + re.escape(END.format(name=name)), re.S)


def sync(root: Path, bodies: dict[str, str], write: bool) -> list[str]:
    drift = []
    for rel, name in TARGETS.items():
        path = root / rel
        text = path.read_text(encoding="utf-8")
        body = bodies[name].strip("\n")           # 只去换行，保留缩进
        m = pattern(name).search(text)
        if not m:
            drift.append(f"{rel}: 缺少 generated:{name} 标记")
            continue
        if m.group(1).strip("\n") != body:
            drift.append(f"{rel}: generated:{name} 已过期")
            if write:
                filled = (BEGIN.format(name=name) + "\n" + body + "\n"
                          + END.format(name=name))
                path.write_text(pattern(name).sub(lambda _: filled, text, count=1),
                                encoding="utf-8")
    return drift


# ---- 结构校验 -------------------------------------------------------------
def validate(groups: list[Group], problems: list[str]) -> list[str]:
    seen, prev = set(), 0
    for g in groups:
        if not g.items:
            continue
        nums = [i.num for i in g.items]
        if nums != list(range(nums[0], nums[-1] + 1)):
            problems.append(f"第{g.num}卷章号不连续：{nums}")
        if prev and nums[0] != prev + 1:
            problems.append(f"第{g.num}卷起始 {nums[0]} 不接上一卷末 {prev}")
        for n in nums:
            if n in seen:
                problems.append(f"第{n}章重复定义")
            seen.add(n)
        if g.range and (nums[0], nums[-1]) != g.range:
            problems.append(f"第{g.num}卷标题写 {g.range}，实际 {nums[0]}—{nums[-1]}")
        prev = nums[-1]
    return problems


def main(argv: list[str]) -> int:
    ap = argparse.ArgumentParser()
    g = ap.add_mutually_exclusive_group()
    g.add_argument("--write", action="store_true")
    g.add_argument("--check", action="store_true")
    args = ap.parse_args(argv[1:])
    root = Path(__file__).resolve().parent.parent

    groups = load_plan(root)
    problems = validate(groups, load_artifacts(groups, root / "正文"))
    bodies = {"status": block_status(groups)}

    if args.write:
        for line in sync(root, bodies, write=True):
            print("已重写", line)
    if args.check:
        problems += sync(root, bodies, write=False)

    print(bodies["status"])
    for p in problems:
        print("-", p)
    return 1 if (problems and args.check) else 0


if __name__ == "__main__":
    raise SystemExit(main(sys.argv))
```

## 检查清单

- [ ] 事实源是文件系统/标题这类“难造假”的输入，不是另一份手写清单
- [ ] 生成块标记成对出现，块内无手写内容
- [ ] 生成器幂等：连跑两次，第二次 0 更新
- [ ] 生成输出与改造前的手写文本事实一致（语气与原句保留）
- [ ] `--check` 覆盖：编号重复、区间不符、卷间断档、文件名与标题不符、孤儿条目、块过期
- [ ] `--check` 接进 publish/commit，失败即停
- [ ] 手改一块能触发退出码 1；`--write` 能复原
- [ ] 规范文档写明“哪个数字不要手改”
- [ ] 下游构建产物统计与生成块一致（改完跑一次构建核对）

## 常见坑

| 坑 | 后果 | 改法 |
|---|---|---|
| `body.strip()` | 吃掉首行缩进，块内排版塌陷 | `strip("\n")` |
| `text.replace(old, new)` 当 old 为空 | 内容插进每个字符之间，文件炸成几十 KB | 正则整块替换 |
| 只改一卷编号 | 下游章号全部错位、出现断档 | 全局平移 + check 兜底 |
| 生成块放在渲染可见位置 | 中文注释出现在页面上 | HTML 注释 `<!-- -->` |
| 手写“下一章见 44”在块外 | 仍然是第二套状态 | 纳入块或去掉数字 |
