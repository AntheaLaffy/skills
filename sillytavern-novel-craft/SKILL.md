---
name: sillytavern-novel-craft
description: 深度整合 SillyTavern（酒馆）底层引擎的小说创作与世界观推演框架。运用酒馆四大核心杀手锏：深度注入(Insertion Depth & A/N)、条件门世界书(Selective Lorebook Gates)、宏变量状态机(Macro State Management)、以及正则导演机位(Regex Director Notes)，打造工业级反吃书、长篇一致性小说辅助引擎。
---

# SillyTavern 工业级小说创作引擎 (sillytavern-novel-craft)

本技能超越普通的“AI 对话/续写 Prompt”，深度汲取 **SillyTavern 底层上下文管理机制与渲染引擎**，解决长篇网文与严肃小说创作中的四大致命痛点：**吃书忘设定、战力膨胀失控、千人一面、以及上下文爆仓导致的智商下降**。

---

## 一、 酒馆底层精髓与落地机制

### 1. 深度注入管道 (Insertion Depth & Position Control)
普通 Web 前端将所有设定堆叠在最前方的 System Prompt，极易被模型注意力机制稀释并遗忘。
* **精髓机制**：通过酒馆的 `@Depth` (例如 `Depth: 3~4`) 将当前篇章的核心目标、战力红线或 Author's Note 强行注入在**紧邻最新上下文上方**。
* **效果**：模型在生成最新段落时拥有最高的遵循权重，确保当前场景的主线不跑偏。

### 2. 条件门世界书系统 (Selective & Logical Lorebook Gates)
拒绝全量载入世界书，使用精准逻辑门控节省 Token 并提高命中精度：
* **Selective 逻辑门（双键 AND 触发）**：
  * 主触发词 `key`: `["拔剑", "杀气", "神通", "太玄剑典"]`
  * 协同从词 `keysecondary`: `["林渊", "主角"]`
  * 仅当主角林渊在进入交手语境时才激活战力约束，日常喝茶闲聊绝不误触发。
* **Sticky & Cooldown (黏性与冷却)**：
  * `sticky: 3`：战斗招式触发后自动在接下来 3 轮上下文保留，保持招式连贯。
  * `cooldown: 6`：退出战斗后 6 轮内不再无故唤醒，避免注意力污染。
* **Prevent Recursion (防死循环)**：防止 A 设定触发 B，B 递归触发 A 造成 Token 暴炸。

### 3. 宏变量状态机 (Macro-driven State Machine)
在推演过程中实时追踪角色物理指标，彻底杜绝“吃书”：
* `{{setglobalvar key=scene_tension value=8}}`：调控当前场景张力刻度（1-10）。
* `{{getglobalvar::chapter_beat}}`：获取当前幕的核心驱动任务。
* 每轮交手或探索后，使用 Slash Command 自动对角色真元、负伤阶梯、持有道具进行结算。

### 4. 正则分层渲染与导演机位 (Regex Director Pipeline)
通过酒馆的 `regex_placement: AI_OUTPUT (2)` 后处理：
* 模型在生成正文前强制输出 `<stage_direction>`（本幕剧情反转点与伏笔设计）。
* 前端通过专属正则自动将其折叠为 `<details>` 导演小贴士，正文区仅保留纯净、无任何 AI 机械味的小说正文。

---

## 二、 核心命令与工作流规范

### 1. 场景推进 (`/novel_advance`)
利用全局张力宏与动态镜头语言推进正文：
```stscript
/setglobalvar key=scene_tension 8
/setglobalvar key=chapter_beat {{pipe::输入本幕推进目标}}
/instruct 请以严苛的小说正文模式紧接前文推进场景：
1. 【导演思维】：在最开端用 <stage_direction>简述此幕核心反转点与伏笔</stage_direction>（将被前端自动折叠）。
2. 【视听具象】：Show, don't tell。通过微表情、肌肉绷紧程度、空气温差与声波共振展现紧张感（当前张力刻度：{{getglobalvar::scene_tension}}/10）。
3. 【战力与状态】：严格尊重人物体内真元与负伤状态，杜绝机械降神。
4. 字数控制在 1000-1500 字，段末抛出悬念钩子。
/gen
```

### 2. 状态机审计与反吃书审查 (`/novel_audit`)
```stscript
/instruct 请静默审查上一轮交手/剧情，更新主角与敌手的状态机：
- 消耗品/真元损耗评估
- 肉体/神魂负伤阶梯（轻微/挫伤/经脉受损/濒死）
- 新获战利品或信息情报
请以精准的变量清单输出，并在最后给出下一回合允许的最大爆发阈值。
```

### 3. 生成双键 Selective 世界书 (`/novel_lorebook`)
```stscript
/instruct 提取上文出现的新名词/招式/角色，生成符合酒馆底层引擎的高级 Selective Lorebook 条目：
{
  "key": ["触发主词（如：拔剑、秘术名）"],
  "keysecondary": ["联动从词（如：林渊、对手名）"],
  "comment": "条目注释与战力归档",
  "content": "[Invariants & Bounds]:\n法则/招式严格规则，包含代价、消耗、视觉特征与禁止事项（100字以内）。",
  "selective": true,
  "sticky": 2,
  "cooldown": 6,
  "depth": 4
}
```

---

## 三、 本地配套资产

以下前三项路径相对于本技能目录。

| 资产类型 | 存放路径 | 作用 |
| :--- | :--- | :--- |
| **技能规范** | `SKILL.md` | 本技能说明文档与工作流守则 |
| **世界书范式** | `worldinfo/SelectiveLorebookExample.json` | 具备 Selective/Depth/Sticky 机制的标准世界书 |
| **正则后处理** | `regex/NovelRegexPresets.json` | 幕后思考链自动折叠与标点规范脚本 |
| **酒馆一键预设** | `SillyTavern/data/default-user/QuickReplies/NovelCraft.json` | 已装配好的酒馆交互式动作面板（4大功能按键） |
