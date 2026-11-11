# Skills

面向 Codex、Claude Code 和其他编程智能体的本地 skill 集合。每个子目录都是一个可以独立加载的 skill，使用时直接读取对应目录中的 `SKILL.md`。

## 内容

- `fframes-video/`：本地安装的视频生成技能；上游通过 `.upstream/fframes` Git submodule 管理。

- [`codemap/`](codemap/)：生成和维护分层 `CODEMAP.md`，按任务路由代码阅读范围。
- [`project-maintenance/`](project-maintenance/)：在功能开发、重构和后端迁移后同步代码地图、实现、测试、构建入口和文档，并处理已退役内容。
- [`state-single-source/`](state-single-source/)：状态设计——把进度/统计/清单类事实收成唯一来源，用脚本生成派生文本并校验漂移，适用于文档工程、小说进度、看板与里程碑；与 `project-maintenance` 分工：后者管地图/退役/契约验证，前者管派生数字。
- [`performance-gradient-optimization/`](performance-gradient-optimization/)：用可复现实验优化延迟、吞吐、CPU、内存、I/O、能耗和成本。
- [`baidu-drive/`](baidu-drive/)：百度网盘命令行操作与本地脚本。
- [`sillytavern-novel-craft/`](sillytavern-novel-craft/)：SillyTavern 小说创作与世界书工作流。
- `research-verification/`：事实调研与来源核查。
- [`the missing semester/`](the%20missing%20semester/)：The Missing Semester 主题的中文学习型 skills 与练习资料。

## 随插件发布的技能

这些技能的正文已经收进对应插件，插件更新即技能更新，本仓库不再保存副本：

| 技能 | 来源 |
| --- | --- |
| `tomato-todo`、`exam-teaching`、`exam-authoring`、`study-planning`、`study-diagnosis`、`study-memory` | `~/code/todo/.agents/worktrees/tomato-ts-rewrite/plugin/skills/`（`@local/tomato-dsh`） |
| `parallel-agent-development` | `~/code/dsh-three-level-agent/skills/` (`dsh-three-level-agent`) |

本机要给 Codex、Claude Code 等非 DSH agent 用同一份内容时，跑对应插件的
`link-skills.mjs`（默认链到 `~/.agents/skills`，链接指向插件包，之后更新同样生效）：

```bash
node ~/code/todo/.agents/worktrees/tomato-ts-rewrite/plugin/link-skills.mjs
node ~/code/dsh-three-level-agent/link-skills.mjs
```

这样本仓库里出现的这 7 个目录是符号链接；它们已写进 `.gitignore`，不入库，换机器时
重新跑上面的命令即可。`codemap`、`project-maintenance`、`state-single-source` 里提到
`parallel-agent-development` 时，指的就是插件包中的那一份。

## 安装

所有 agent 的用户级 skills 目录统一链接到本仓库，例如：

```bash
ln -s "$HOME/code/skills" "$HOME/.codex/skills"
ln -s "$HOME/code/skills" "$HOME/.agents/skills"
ln -s "$HOME/code/skills" "$HOME/.claude/skills"
ln -s "$HOME/code/skills" "$HOME/.gemini/skills"
ln -s "$HOME/code/skills" "$HOME/.config/opencode/skills"
```

上述命令要求目标路径不存在；已有目录须先合并内容并移开，避免创建嵌套链接。其他 agent 同样将其用户级 skills 路径直接链接到这里，不再维护独立副本。反过来，随插件发布的技能由插件的 `link-skills.mjs` 链接进来，见上一节。

`the missing semester/` 保留课程源码结构，根目录的相对符号链接让只扫描一级目录的工具也能发现这些 skills。`.system/` 是 Codex 管理的内置 skills，不纳入版本控制。插件内置 skills 与各项目专属 skills 仍由各自来源管理。

`exam-authoring/references/local-library.md` 与 `tomato-todo/references/local-context.md` 是本机导航，保留在本机且已忽略，不随公开仓库发布；克隆到新机器时按实际资料和连接自行建立。

## fframes 上游技能

Git 仓库只记录 `.upstream/fframes` 子模块的上游地址和提交版本，不重复提交技能副本。本地 `fframes-video/` 是实际安装的技能目录，已加入 `.gitignore`，各 agent 通过现有 skills 目录链接共用它。

新机器初始化子模块并安装技能：

```bash
git submodule update --init --depth 1 .upstream/fframes
cp -a .upstream/fframes/skills/fframes-video ./fframes-video
```

更新上游并同步本地技能：

```bash
git -C .upstream/fframes fetch origin main
git -C .upstream/fframes checkout --detach origin/main
cp -a .upstream/fframes/skills/fframes-video/. ./fframes-video/
git add .upstream/fframes
```

更新前保存本地自定义改动，并核对上游删除的文件；复制命令不会清除旧文件。审查后提交新的子模块指针，其他机器用 `git submodule update --init` 获取该版本，再同步本地技能。可选用 `git -C .upstream/fframes sparse-checkout set skills/fframes-video` 减少检出的文件。

## 维护约定

代码、接口、命令、目录、配置、平台支持或用户流程改变时，同步受影响的 README、架构说明、示例、导航入口和交叉链接。功能退役时，先核对反向链接和共享资源，再删除只服务于旧功能的文档、示例、截图、fixture、导航项及失效链接。

`project-maintenance/scripts/maintenance_state.py` 可以记录审查范围并检测后续的代码、测试、配置、文档和代码地图变化：

```bash
python3 project-maintenance/scripts/maintenance_state.py inspect --root <project-root>
python3 project-maintenance/scripts/maintenance_state.py record --root <project-root> --reviewed
python3 project-maintenance/scripts/maintenance_state.py check --root <project-root>
```

## 来源与许可

本仓库不对所有目录声明统一许可证。请按照每个 skill 自带的许可证和来源说明使用：

- `codemap/` 保留上游 MIT 许可证文件。
- `the missing semester/` 中的资料保留 MIT The Missing Semester 的 CC BY-NC-SA 4.0 改编说明和链接。
- 其他目录的许可范围以其自身文件和来源约定为准；没有单独许可声明时，不要推断为可任意再授权。
