---
name: parallel-agent-development
description: >
  用 dsh 三级代理并行开发：主代理按互不重叠的文件划分任务，二级代理各自 git worktree 分叉隔离改动、完成后合并；三级及更深可在同一 worktree 内协作或再分叉；并说明上下级/兄弟代理的通信机制（subagent/subagent_fork/agent_message/send_message/list_agents/interrupt_agent）。适用于并行化大改动，或决定各委托层级的 worktree 隔离与消息路由。
  Use when parallelizing a large change across multiple subagents in DeepSeek Harness, deciding git worktree isolation per delegation depth, or wiring parent-child agent communication.
---

# 并行代理开发

## 为什么并行

`project-maintenance`、`state-single-source`、`codemap` 这类技能会反复读写同一批文件（地图、状态、文档）。串行同步时，每个 agent 的中间态都要来回搬运，同步时间会极大拖慢开发。把它们拆成互不重叠的并行子任务、各自在独立 worktree 里跑，是唯一能压住开销的方式。

## 三级代理层级

dsh 的委托深度最多 2：主代理(depth 0) → 子代理(depth 1) → 子子代理(depth 2)。对应「三级」：

| 你口中的级 | dsh depth | 谁创建 | 隔离要求 |
|---|---|---|---|
| 一级 / 顶层 | 0 | 主代理（本会话） | 拥有主工作区 |
| 二级 | 1 | 主代理用 `subagent` / `subagent_fork` | 必须各自 `create_worktree` 分叉 |
| 三级（及更深） | 2 | 二级代理再 `subagent` | 可共享父 worktree，也可再分叉 |

> dsh 最多三级（depth 2）。「更高级别」在 dsh 里到三级为止；规则可推广——二级以下全部共享或再分叉，但绝不允许二级直接写主工作区。

## 二级代理：必须 worktree 分叉

二级代理直接改主工作区会把未完成的中间态混进主分支。每个二级代理先 `create_worktree` 从主分支（或某提交）切出新分支，再在其 cwd 里开发、提交；主代理最后合并。

- 一个二级代理一个 worktree + 一个分支，只动自己那份互不重叠的文件。
- worktree 内自由 add/commit，不污染主工作区；未提交文件留在源 checkout，`create_worktree` 不带走。
- 完成后主代理 `git merge` / cherry-pick 回主分支，再删 worktree；冲突只发生在合并点，不在开发途中。

## 三级及更深：可共享，也可再分叉

三级代理跑在某个二级代理的 worktree 里，最坏只弄脏这个 worktree、碰不到主工作区，所以允许共享同一 worktree 相互「弄脏」。任务也互不重叠且量大时，同样可以各自再 `create_worktree` 隔离。

选共享还是再分叉，看三点：三级任务是否互不重叠、改动文件是否冲突、最终由谁合并。共享时由父代理（二级）在合并前整理干净；再分叉时三级各自提交、二级负责合并。

## 收口顺序

1. 主代理按互不重叠的文件划分任务，为每个二级代理 `create_worktree`。
2. 一批并行下发 `subagent`（同一轮一起发），各自在 worktree 内开发 + 提交。
3. 二级代理需要时再往下拆三级（共享或再分叉）。
4. 全部结束后主代理逐个 `git merge`，处理冲突，删 worktree。
5. 跑 `project-maintenance` / `state-single-source` / `codemap` 的校验脚本收口。

## 上下级代理通信机制

### 创建与投递

- `subagent(prompt, cwd, provider, model, ...)`：开一个**独立的二级代理**，立即返回 id，不共享本会话上下文。`cwd` 指到它的 worktree。
- `subagent_fork(prompt, cwd)`：开一个**继承本会话已完成轮次**的二级代理，适合续接当前上下文的分析/审查。
- `agent_message(agent_id, message)`：给委托树里**任意**代理发消息，agent_id 用 `list_agents` 里的 id，或快捷名 `"parent"`（直属上级）、`"root"`（主代理）。异步：消息进收件人 FIFO 队列，后台加锁投递，失败回报——不要重试，继续干活。
- `send_message(agent_id, message)`：只给**直属可继续的子代理**发消息（作为常驻子代理时也可给直属父代理发）。工作中的代理在下一步收到；空闲代理立即开新轮。返回投递确认，**不是答案**。

### 汇报与收口

- 子代理完成，用 `send_message` 把结果回报直属父代理；父代理继续对话也用 `send_message`。
- 子代理要找更上级，用 `agent_message(agent_id: "parent")` 或 `agent_message(agent_id: "root")`。
- `list_agents("children")` 列直属子代理（任意状态都可 `send_message`）；`list_agents("descendants")` 列整棵子树（depth>1 的只接受 `interrupt_agent`）。
- `interrupt_agent(agent_id)`：让某子代理停手。直属子代理之后还能 `send_message` 继续对话；更深的子孙外部执行永久停止、不能续。

### 通信纪律

- 父子长对话用 `send_message`；跨级、兄弟之间用 `agent_message`，消息要自包含（收件人看不到你的上下文）。
- 不要轮询/空等子代理：runtime 会在其结束时通知你；等结果时结束自己的轮次。
- 兄弟默认不经主代理互相轰炸；需要协作就把必要上下文一并放进消息。

## 反模式

- 二级代理直接写主工作区（弄脏工作区，合并困难）
- 多个并行代理改同一文件（必然冲突）
- 空轮询子代理状态（runtime 会通知，等就行）
- 用 `agent_message` 给直属子代理做需续写的长对话（用 `send_message`）
- 没等全部子代理结束就合并（部分结果缺失）
