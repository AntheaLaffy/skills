# Codex conversation

## User

注意整个流程是自动化(工具)+智能化(agent)，已达到高效增删改查，节省token，你探索一下合适的工具(或者动态自建工具)和方式？

## Assistant

我会先读这个仓库的维护技能说明和目录结构，判断当前技能工作流里哪些步骤反复依赖人工或占用上下文，再据此挑选自动化工具和 agent 分工方式。

## Activity

    $ cat /home/fuurin/.codex/skills/project-maintenance/SKILL.md && rg --files -g 'AGENTS.md' -g '*.md' -g '*.py' -g '*.sh' -g '*.json' -g '*.toml' | head -200
    ---
    name: project-maintenance
    description: Keep a project's code maps, implementations, tests and build routes current after feature work, refactoring, API changes or backend migrations. Use for maintenance and retirement of superseded code in assembly, Rust, Python, TypeScript, C and C++ projects, especially native UI and dedicated inference engines.
    ---
    
    # Project Maintenance
    
    把维护作为当前开发任务的完成条件。同步受影响的地图，清除已被替代的实现和失效测试，验证仍然承诺的产品行为。对用户已经授权的任务内维护直接执行；跨出任务范围、改变支持平台或对外接口时，先给出具体影响。
    
    ## 确认当前契约
    
    - 从指定项目根目录开始，读取 `AGENTS.md`、当前需求及已有地图。保留规定的语言、渲染后端、推理引擎、平台支持和源码所有权边界。
    - 检查已提交、暂存、未暂存及新增文件。区分本次任务和其他人的未完成工作，不替别人整理不相关改动。
    - 新项目随首个可运行实现建立地图；已有项目以当前源码验证地图，不以旧文档推断不存在的接口。
    - 链接进 UI 仓库的引擎源码、子模块和独立 checkout 各自验证真实目录；一个仓库的记录不证明另一个源码树已经同步。
    
    Git 项目可先运行只读检查：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    ```
    
    脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    
    ## 随改动更新地图
    
    ### 双射性与增删改
    
    每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
    
    - **增**：补上当前契约所需的实现、有效验证和遗漏的地图条目。
    - **删**：删除已确认冗余或退役的代码，以及其专属测试、依赖、构建路线、文档和地图条目。
    - **改**：优先原地修正仍承担职责的实现，同步职责、接口、依赖、路径和行范围；不靠另加平行版本绕过旧问题。
    
    双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”。按 `$codemap` 格式，每个文件在本层 Files 中恰好一条，每个直接子目录在 Subdirectories 中恰好一条；其他格式明确等价的归属。纯重导出文件也不能因导出表存在而漏记。任务入口、符号表和依赖边允许多处指向同一实体，不构成重复归属；不要求每个函数或使用场景各有一张表。索引自身不递归作为源码索引。
    
    以当前工作树双向核对：源码到地图查缺漏和重复归属，地图到源码查幽灵路径和失真的职责、接口、依赖及行范围。明确排除范围，不能通过扩大忽略规则掩盖缺漏。完成时简要说明增删改的处理或不适用依据，以及仍未闭合的缺口。
    
    双射只约束声明的索引范围。根据用户关注点和项目约定，不关心的部分可以明确忽略或只保留目录边界，无需逐文件维护。必须在根地图的“范围与忽略项”中记录 `根目录相对路径/模式 | 处理方式 | 原因/依据`；处理方式为“排除”或“仅保留边界”。后者保留唯一目录条目并标注“内部未索引”，不要求内部文件表或子地图。常规忽略项可按默认规则、`.gitignore` 等明确来源归组，项目特有忽略项及重新纳入的例外必须明确列出。根地图是范围的权威记录，其他位置只引用。
    
    忽略索引不等于过时、可删除或免于正确性验证。后续任务触及忽略区或依赖其契约时，读取所需代码并重新评估范围；决定变化时同步标记，不能把未索引内容宣称为已核对。维护脚本的扫描排除与地图的索引排除分别服务于变更检测和导航，不能未经判断直接互相套用。
    
    使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    
    - 文件增删、移动、重命名：同步本层文件表、父目录路由及任务入口，移除已不存在目录的地图。
    - 接口或依赖变化：同步导出符号、调用入口、跨目录依赖及消费者。
    - 汇编与 FFI：记录真实符号、调用约定、结构布局和偏移、内存所有权、回调线程，以及 C/Rust 包装层；用头文件、声明、调用点和链接符号核对。
    - UI 与推理引擎：记录事件到状态到渲染/推理的路径；模型边界写明 tensor shape、dtype、布局、量化格式和必要的转换。
    - 实现变化：若职责、数据流、契约或大文件功能行范围变化，仍然更新相关说明；纯内部等价修改可以不改地图，并说明判断依据。
    
    地图保留一份当前版本，原地更新。只在 agent 规则文件中放简短入口和完成条件，避免复制整份地图。
    
    ## 文档一起更新和退役
    
    - 代码、接口、命令、目录、配置、支持平台或用户流程变化时，检查 README、架构说明、运行手册、示例、任务入口和交叉链接；把仍然有效的说明改成当前行为。
    - 功能、接口或构建路线退役时，删除只服务于它的文档、示例、截图、fixture、导航条目和失效链接；先核对反向链接与仍在使用的共享资源。Git 历史承担已删除内容的留存。
    - 保留一份权威说明，合并重复文档，避免用过时的 `old/`、`v1/`、`backup/` 文档树掩盖当前状态。文档删除必须有当前契约或调用关系依据，不能只因文件很旧或无人引用。
    - 完成维护后，至少检查受影响 Markdown 链接、命令、路径和状态描述；不能验证的外部链接或平台行为要明确记录。
    
    ## 实现与测试一起退役
    
    **时效淘汰性**：每次变更主动复查受影响旧实现和临时层的保留依据、迁移状态与移除条件。替代实现已验证、消费者已迁移或旧需求已撤销时，在本次任务内完成退役；不能仅在地图上标成 deprecated 后无限保留。暂时仍需保留的分支必须记录具体消费者或当前契约及可检查的移除条件，在后续相关改动时重新核对。时效由当前需求和迁移证据判定，不按文件年龄或任意期限机械删除。
    
    对候选项先核对用途、调用者、构建目标、平台和公开契约，再在本次授权范围内处理：
    
    - 已替换并完成迁移的实现：清除旧代码、专属包装层、构建入口、依赖、文档、测试与 fixture；历史由 Git 保留，不复制 `old/`、`v1/`、`backup/` 源码树。
    - 临时适配层：迁移完成就移除；仍有使用者时，写清当前消费者和具体移除条件。
    - 重复或仅验证 mock 自身的测试：合并或删除，保留不同输入、平台、边界条件和已发现故障的有效断言。
    - 测试取消或改变的需求：按当前契约退役或重写，同时核对相关 fixture、runner 配置和文档。
    - 失败、被跳过、年代久远、覆盖率低或图中没有入边，都不是单独的删除依据。先区分产品回归、环境问题、有效平台测试和真正失效的需求。
    - 新测试只验证本次行为、接口或实际故障，避免把当前实现细节再写一遍。对仍有效的回归保持独立检测能力；不能为使检查通过而删除断言、改宽误差或重录错误的快照。
    
    外部导出、动态注册、函数指针、FFI、汇编入口和跨平台分支需要额外核对消费者。只证明“无本地文本引用”时，报告剩余不确定性，继续完成其余可确认的维护。
    
    ## 验证并记录当前状态
    
    按 [验证与退役判据](references/verification.md) 只选择本次涉及的语言、平台、UI 或推理验证。优先复用现有验证入口；不要为了使用这个 skill 重建一套重复测试框架。
    
    完成本次维护和有效验证后，在原地替换单份审查记录：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py record --root <project-root> --reviewed
    python3 <skill-dir>/scripts/maintenance_state.py check --root <project-root>
    ```
    
    `--reviewed` 表示 agent 已实际审查当前范围并完成应做的验证，不是用户批准，也不能替代验证。未完成的关联改动、未知失败或未验证的其他源码树必须如实报告；不能为让检查变绿而重录。
    
    记录位于 `.project-maintenance/state.json`，包含当前文件哈希与审查版本。`check` 在文件或审查范围变化后返回非零，供已有验证命令、pre-commit 或 CI 调用；新增这些集成时使用项目现有方式，编辑一个可重复更新的入口。脚本的范围、排除模式和能力限制见验证参考。
    
    最终报告本次同步的地图、退役项及其依据、实际验证结果和未解决问题。指标是当前契约和可维护性，不是删除数量、测试数量或通过率。
    SKILL.md
    scripts/maintenance_state.py
    references/verification.md
    ✓ • 0ms

## User

继续

## Assistant

继续。我会把“工具提取事实、agent 做语义判断、按需读取证据”落实到这个维护技能里，先检查现有脚本能复用什么，再选择值得补上的自动化能力。我会使用 skill-creator 来组织这次技能更新。

## Activity

    $ for path in /AGENTS.md /home/AGENTS.md /home/fuurin/AGENTS.md /home/fuurin/code/AGENTS.md /home/fuurin/code/skills/AGENTS.md /home/fuurin/code/skills/project-maintenance/AGENTS.md; do if [ -f "$path" ]; then printf '\n%s\n' "$path"; cat "$path"; fi; done
    ✓ • 0ms

## Activity

    $ pwd; git status --short; rg --files --hidden -g '!/.git' -g '!*.pyc' -g '!__pycache__' .
    /home/fuurin/code/skills/project-maintenance
     M ../codemap/SKILL.md
     M SKILL.md
    ./references/verification.md
    ./scripts/maintenance_state.py
    ./agents/openai.yaml
    ./SKILL.md
    ✓ • 0ms

## Activity

    $ cat /home/fuurin/.codex/skills/.system/skill-creator/SKILL.md; cat /home/fuurin/.claude/skills/agent-context-management/SKILL.md; cat /home/fuurin/.codex/skills/find-docs/SKILL.md
    ---
    name: skill-creator
    description: Create or update a Codex skill with appropriately scoped instructions and any needed supporting resources.
    metadata:
      short-description: Create or update a skill
    ---
    
    # Skill Creator
    
    Create skills that give Codex useful, non-obvious guidance without constraining unrelated work.
    
    ## Core Principles
    
    **Assume Codex is already capable.** Include only information that changes its decisions or improves its work. Remove generic advice, repeated instructions, speculative edge cases, and examples that do not materially clarify the task.
    
    **Preserve user intent and scope.** A skill should support the requested task, not replace the user's chosen product, expand the assignment, modify unrelated configuration, or imply permission for additional external actions. Do not turn a particular example, past failure, or personal preference into a universal requirement.
    
    Approval to complete a task does not expand its scope or execution permissions. For retrying or externally mutating workflows, define a stopping condition proportional to the risk.
    
    **Match specificity to the risk.** Give the model room to choose an appropriate approach when multiple approaches are reasonable. Use detailed steps, deterministic scripts, or absolute language only when correctness, safety, permissions, or a genuinely fragile workflow requires them.
    
    For open-ended work, describe the outcome and relevant decision criteria. For workflows with a preferred shape, offer useful examples or configurable scripts. Reserve fixed sequences and narrow parameters for operations where deviation would cause a concrete problem. Preserve non-obvious operational invariants, distinguish actual requirements from optional recommendations or local conventions, and avoid restating policies already enforced elsewhere.
    
    **Keep discovery cheap and precise.** Skill names and descriptions are available before a skill is loaded. Describe the actual capability and when it applies, adding exclusions only when they prevent likely misrouting. Avoid exhaustive capability lists and catchalls that attract unrelated requests.
    
    Keep skills self-contained; refer to another skill or tool only when the requested workflow genuinely requires it and it is available in the target environment. Specialized review, hardening, or audit workflows should apply when requested or genuinely needed, not merely because ordinary work touches the same subject.
    
    **Disclose detail progressively.** Keep shared purpose, essential constraints, and useful routing in `SKILL.md`. Put substantial mode-specific guidance, schemas, examples, or procedures in supporting references and read only the references relevant to the current task. A simple self-contained skill does not need a router or extra files.
    
    ## Anatomy of a Skill
    
    Every skill is a folder containing a required `SKILL.md` file and any optional resources its actual workflow needs:
    
    ```text
    skill-name/
    |-- SKILL.md                 Required skill instructions
    |   |-- YAML frontmatter     Required name and description
    |   `-- Markdown body        Instructions loaded when the skill is used
    |-- agents/                  Optional UI metadata and invocation policy
    |   `-- openai.yaml
    |-- scripts/                 Optional executable helpers
    |-- references/              Optional documentation loaded as needed
    `-- assets/                  Optional files used in generated output
    ```
    
    Choose the structure that fits the actual task. Some skills are short and self-contained; others route among operating modes or delegate complex mechanics to scripts. Avoid creating directories, placeholders, examples, or ancillary documentation without a clear use.
    
    ### SKILL.md
    
    The YAML frontmatter identifies the skill and determines when it should be considered. Include the required `name` and `description`, and preserve supported optional fields such as existing `metadata` when appropriate.
    
    The Markdown body is loaded only when the skill is used. Put the purpose, essential workflow, real constraints, and useful links there. Keep detailed procedures and examples in supporting references when they are relevant only to particular modes.
    
    Skill information is disclosed in three stages:
    
    1. **Name and description:** Available during skill selection, so keep them concise and discriminating.
    2. **SKILL.md body:** Loaded when the skill applies, so keep its instructions relevant to that task.
    3. **Supporting resources:** Read or execute only when the current task actually needs them.
    
    The entrypoint should be as short as the task permits while retaining important constraints. A large upper bound is not a target: move conditional detail into references when doing so improves clarity or context use, rather than waiting for the file to become unwieldy.
    
    ### Scripts
    
    Use `scripts/` for executable code when the same logic would otherwise be rewritten repeatedly or deterministic execution materially improves reliability.
    
    - **Example:** `scripts/rotate_pdf.py` for a PDF operation that would otherwise require recreating the same code.
    - **Useful for:** Repeated transformations, reliable API operations, data processing, and other concrete automation.
    - **Validation:** Run new or changed scripts to verify their behavior. Scripts can usually be executed without loading their full implementation into context, although an agent may need to inspect them when patching or adapting them.
    
    ### References
    
    Use `references/` for documentation that is needed only in particular contexts.
    
    - **Examples:** `references/schema.md` for database tables, `references/policies.md` for domain rules, `references/api_docs.md` for an API, or separate writing guides for different deliverables.
    - **Useful for:** Schemas, API documentation, company policies, format-specific procedures, detailed workflows, and substantial examples.
    - **Routing:** Link each reference from `SKILL.md` or another relevant resource and explain when it should be read. Keep information in one place instead of duplicating it across the entrypoint and references.
    
    Keep references focused on maintained, task-specific information that changes the agent's decisions. Avoid copied manuals, exhaustive catalogs, and generic tutorials already available from authoritative sources. Before removing existing resources, inspect their callers and purpose.
    
    For large references, include useful search terms or a short contents section when that makes the needed material easier to find.
    
    ### Assets
    
    Use `assets/` for files that belong in generated output rather than in the model's instructions.
    
    - **Examples:** `assets/logo.png`, `assets/slides.pptx`, `assets/font.ttf`, or `assets/frontend-template/`.
    - **Useful for:** Templates, images, fonts, icons, boilerplate projects, and other files copied or adapted into the result.
    - **Context:** Do not load assets as instructions unless the task requires inspecting them.
    
    ### UI Metadata and Invocation Policy
    
    `agents/openai.yaml` can provide UI-facing metadata such as `display_name`, `short_description`, and `default_prompt`, along with invocation policy. When creating or updating those settings, read [references/openai_yaml.md](references/openai_yaml.md) and keep the values consistent with the skill.
    
    Automatic skill selection is allowed by default. Change that default only when the user explicitly requests an explicit-only skill:
    
    ```yaml
    policy:
      allow_implicit_invocation: false
    ```
    
    This keeps the skill available when explicitly invoked as `$skill-name` without adding it to the model context automatically. Preserve unrelated existing UI, policy, and dependency fields when updating `agents/openai.yaml`.
    
    The initializer creates this file automatically. For new or interface-only metadata, generate it with:
    
    ```bash
    scripts/generate_openai_yaml.py <path/to/skill-folder> --interface key=value
    ```
    
    The generator replaces the entire file. If an existing file contains `policy` or `dependencies`, update only the intended fields in place instead of regenerating it.
    
    Include optional interface fields only when the user provides or requests them.
    
    ### What Not to Include
    
    Include files that directly support the skill's work. Avoid adding a `README.md`, installation guide, changelog, duplicated quick reference, or other auxiliary documentation unless a specific task or packaging requirement calls for it.
    
    ## Progressive Disclosure in Practice
    
    For a skill with multiple substantial modes, keep the shared guidance and mode-selection criteria in `SKILL.md`. Link each supporting reference where its use becomes relevant. Do not load every reference by default, duplicate reference content in the entrypoint, or add a routing layer when there is nothing meaningful to route.
    
    For example, a deployment skill can keep provider selection in `SKILL.md` and separate provider details:
    
    ```text
    cloud-deploy/
    |-- SKILL.md
    `-- references/
        |-- aws.md
        |-- gcp.md
        `-- azure.md
    ```
    
    When the user chooses AWS, read `references/aws.md`; do not also load the GCP and Azure guides. The same pattern can separate business domains, deliverable types, or other genuinely distinct operating modes.
    
    A short skill can instead route to details only when an advanced operation needs them:
    
    ```markdown
    ## Documents
    
    Handle ordinary edits directly.
    
    - For tracked changes, read [references/redlining.md](references/redlining.md).
    - For document internals, read [references/ooxml.md](references/ooxml.md).
    ```
    
    These examples illustrate options, not a required structure. Choose the organization that makes the skill easier to use without loading irrelevant material.
    
    ## Create or Update a Skill
    
    Adapt the work to the request. Creating a complex new skill may involve understanding realistic use cases, choosing supporting resources, initializing files, writing instructions, and validating the result. A narrow update to an existing skill may require only a focused edit and validation.
    
    Ask clarifying questions only when the missing information matters and cannot be reasonably inferred. Respect a user-specified location; otherwise create discoverable skills in `$CODEX_HOME/skills`, or `~/.codex/skills` when `CODEX_HOME` is unset.
    
    Keep automatic skill selection enabled unless the user explicitly requests an explicit-only skill. When the intended invocation mode is genuinely unclear and matters to the requested workflow, ask whether the user wants normal automatic discovery or explicit-only invocation; otherwise preserve the default. Do not infer explicit-only invocation from sensitive operations or required approvals: keep the skill discoverable and require authorization immediately before the actual mutation. Preserve an existing skill's invocation policy unless the user asks to change it.
    
    For a new or substantially revised skill, consider the actual requests it should handle and which reusable resources would improve those tasks:
    
    - A repeated PDF transformation may justify a `scripts/rotate_pdf.py` helper.
    - An application-building workflow may benefit from an `assets/frontend-template/` starter.
    - A data-analysis skill may need a `references/schema.md` guide to avoid rediscovering table relationships.
    
    Create those resources only when their concrete benefit justifies them. If the user has already explained the task clearly, proceed without requesting additional examples.
    
    ### Naming
    
    - Use lowercase letters, digits, and hyphens.
    - Keep names under 64 characters and prefer short action-oriented names.
    - Namespace by tool or domain when doing so improves discovery.
    - Name the skill folder after the skill.
    
    ### Initialize a New Skill
    
    For a new skill, use the bundled initializer when it helps create the required files consistently:
    
    ```bash
    scripts/init_skill.py <skill-name> --path <output-directory> [--resources scripts,references,assets] [--examples]
    ```
    
    For example:
    
    ```bash
    scripts/init_skill.py my-skill --path "${CODEX_HOME:-$HOME/.codex}/skills"
    scripts/init_skill.py my-skill --path "${CODEX_HOME:-$HOME/.codex}/skills" --resources references
    ```
    
    Request only the resource directories the skill needs. Use `--examples` only when concrete placeholders would help, and replace or remove them before finishing. Do not initialize an existing skill again.
    
    The initializer creates the skill directory, a concise `SKILL.md` starter, and `agents/openai.yaml`. It creates resource directories and example files only when requested. Pass generated UI values as `--interface key=value` when needed.
    
    ### Write the Instructions
    
    The frontmatter `description` should briefly explain what the skill does and when it applies. Include a meaningful boundary when similar requests should not activate the skill.
    
    For example:
    
    ```yaml
    description: Create or edit Word documents when formatting, tracked changes, or comments require document-specific handling.
    ```
    
    Put detailed workflows, tool choices, examples, and operating modes in the body or relevant references rather than listing them all in the description. Preserve supported optional frontmatter, such as existing `metadata`, when appropriate.
    
    Write only the instructions needed for another Codex instance to perform the task well. State the desired outcome, non-obvious context, real constraints, and relevant references or tools. Preserve the user's explicit choices and existing authorization boundaries. Avoid prescribing a fixed structure, process, or number of steps when the task does not require one.
    
    ### Validate and Iterate
    
    Validate the completed skill with:
    
    ```bash
    scripts/quick_validate.py <path/to/skill-folder>
    ```
    
    The validator checks frontmatter, naming, and unfinished scaffold placeholders; it does not prove that the skill makes good decisions. Also check that descriptions remain discriminating, instructions preserve user intent, references are discoverable, and any added scripts actually work.
    
    When testing is warranted, verify observable behavior or meaningful invariants. Avoid tests that merely match generated wording, headings, or regex patterns.
    
    Improve the skill based on real usage or demonstrated failures. Prefer a narrow correction to accumulating universal rules for every observed example.
    
    ## Independent Forward-Testing
    
    Use an independent subagent pass when a skill is sufficiently complex or risky that realistic behavioral validation would add meaningful confidence, and when delegation is available and authorized. Ordinary creation or small edits do not automatically require subagents.
    
    Give the evaluating agent a realistic user request, the skill, and the minimum raw artifacts needed to perform the task. Do not provide the intended answer, suspected bug, proposed fix, or prior conclusions unless the evaluation genuinely requires them.
    
    For example:
    
    ```text
    Use $skill-name at /path/to/skill-name to complete this realistic request.
    ```
    
    Keep the evaluation scoped to permitted resources and side effects. Use an isolated temporary workspace for generated artifacts so they do not enter the working tree or contaminate later evaluations. Ask for approval when the proposed evaluation would require additional authorization, affect a live production system, or impose substantial time or cost. Review the actual outcome and artifacts, then make only changes supported by the observed behavior.
    ---
    name: agent-context-management
    description: >
      智能体上下文与编排：清空/回退/压缩上下文、llms.txt、AGENTS.md/CLAUDE.md、Skills 按需加载、
      子智能体、并行智能体与 git worktrees、MCP。Use when the user asks to write an AGENTS.md or
      CLAUDE.md, create a skill or subagent, or manage agent context. 触发于「写 AGENTS.md/建 skill/子智能体/管上下文/MCP」。
    ---
    
    # 智能体上下文管理
    
    主线：LLM 上下文窗口有限，且上下文越大性能往往越差（即使没溢出）。框架会自动提供并管理一部分上下文，但大头在你：给智能体它需要的、砍掉不需要的。Skills、子智能体这些机制本质都是「上下文经济」——本仓库本身就是一组 skills 的实践。
    
    ## 上下文六件套
    
    - **清空窗口**：不相关的查询开新对话——最基本的控制。
    - **回退**：撤销对话历史里的步骤；有时比发消息引导转向更省上下文。
    - **压缩（compaction）**：对话过长时自动用 LLM 总结前半段、以摘要替换；部分工具可手动触发。
    - **llms.txt**：给 LLM 读的文档标准（如 cursor.com/llms.txt、ai.pydantic.dev/llms.txt）——每 token 信息密度远高于抓 HTML；依赖在模型知识截止日期之后发布时尤其有用。
    - **AGENTS.md**：智能体启动时整份预填进上下文的说明文件（Claude Code 找 CLAUDE.md）。放跨会话的通用建议：改代码后跑类型检查、怎么跑单测、可浏览的三方文档链接；可用 `/init` 类命令自动生成。
    - **Skills**：AGENTS.md 永远整份加载，Skills 加了一层间接性避免膨胀——给智能体「技能名 + 描述」，它按需打开才占上下文。三者取舍：AGENTS.md 放「永远该知道」的少数规则；Skill 放「按任务才需要」的知识；子智能体隔离「大量中间过程」。
    
    ## 子智能体与并行
    
    - 子智能体：为特定工作流定义的智能体。顶层调用子智能体——顶层上下文不被子智能体看到的一切撑爆，子智能体也只拿自己任务需要的上下文。例：网页研究做成子智能体（查询、搜索、检索、分析、回答案），顶层上下文不被所有检索页膨胀。Claude Code 用 `/agents` 从简短提示生成；dsh 同样有 subagents 机制。
    - 并行智能体：智能体可能在一个问题上干几十分钟——同时跑多个实例：同一任务多跑几次取最佳（LLM 有随机性），或分头做互不重叠的功能；用 git worktrees（见 `git-cli`）隔离彼此的改动。
    - MCP（Model Context Protocol）：连接智能体与工具的开放协议——Notion MCP 服务器让「读 {Notion 文档} 里的规范、起草实现计划、实现原型」成为可能；用 Pulse、Glama 等目录发现 MCP。
    - 可复用提示词：曾作为独立功能存在；在 Codex、Claude Code 中已被 Skills 覆盖——直接写成 skill 即可。
    
    ## 练习
    
    学习材料在 `exercises.md`；用 DeepSeek Harness（dsh）实操的专属练习见 `exercises-dsh.md`。
    
    > 改编自 MIT The Missing Semester 课程 Lecture 7: Agentic Coding（讲义 + 口播稿，CC BY-NC-SA 4.0）：https://creativecommons.org/licenses/by-nc-sa/4.0/ · 课程站点：https://missing.csail.mit.edu/ · 讲座视频：https://www.youtube.com/watch?v=sTdz6PZoAnw
    ---
    name: find-docs
    description: >-
      Retrieves up-to-date documentation, API references, and code examples for any
      developer technology. Use this skill whenever the user asks about a specific
      library, framework, SDK, CLI tool, or cloud service — even for well-known ones
      like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. Your
      training data may not reflect recent API changes or version updates.
    
      Always use for: API syntax questions, configuration options, version migration
      issues, "how do I" questions mentioning a library name, debugging that involves
      library-specific behavior, setup instructions, and CLI tool usage.
    
      Use even when you think you know the answer — do not rely on training data
      for API details, signatures, or configuration options as they are frequently
      outdated. Always verify against current docs. Prefer this over web search for
      library documentation and API details.
    ---
    
    # Documentation Lookup
    
    Retrieve current documentation and code examples for any library using the Context7 CLI.
    
    Run commands with `npx ctx7@latest` so setup always uses the latest CLI without a global install:
    
    ```bash
    npx ctx7@latest library <name> "<query>"
    npx ctx7@latest docs <libraryId> "<query>"
    ```
    
    Optionally install globally if you prefer a bare `ctx7` command:
    
    ```bash
    npm install -g ctx7@latest
    ```
    
    ## Workflow
    
    Two-step process: resolve the library name to an ID, then query docs with that ID.
    
    ```bash
    # Step 1: Resolve library ID
    npx ctx7@latest library <name> "<query>"
    
    # Step 2: Query documentation
    npx ctx7@latest docs <libraryId> "<query>"
    ```
    
    You MUST call `library` first to obtain a valid library ID UNLESS the user explicitly provides a library ID in the format `/org/project` or `/org/project/version`.
    
    IMPORTANT: Do not run these commands more than 3 times per question. If you cannot find what you need after 3 attempts, use the best result you have.
    
    ## Step 1: Resolve a Library
    
    Resolves a package/product name to a Context7-compatible library ID and returns matching libraries.
    
    ```bash
    npx ctx7@latest library React "How to clean up useEffect with async operations"
    npx ctx7@latest library "Next.js" "How to set up app router with middleware"
    npx ctx7@latest library Prisma "How to define one-to-many relations with cascade delete"
    ```
    
    Use the official library name with proper punctuation (e.g., "Next.js" not "nextjs", "Customer.io" not "customerio", "Three.js" not "threejs"). If results look wrong, try alternate spellings such as `next.js` before changing the query.
    
    Always pass a `query` argument — it is required and directly affects result ranking. Use the user's intent to form the query, which helps disambiguate when multiple libraries share a similar name. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.
    
    ### Result fields
    
    Each result includes:
    
    - **Library ID** — Context7-compatible identifier (format: `/org/project`)
    - **Name** — Library or package name
    - **Description** — Short summary
    - **Code Snippets** — Number of available code examples
    - **Source Reputation** — Authority indicator (High, Medium, Low, or Unknown)
    - **Benchmark Score** — Quality indicator (100 is the highest score)
    - **Versions** — List of versions if available. Use one of those versions if the user provides a version in their query. The format is `/org/project/version`.
    
    ### Selection process
    
    1. Analyze the query to understand what library/package the user is looking for
    2. Select the most relevant match based on:
       - Name similarity to the query (exact matches prioritized)
       - Description relevance to the query's intent
       - Documentation coverage (prioritize libraries with higher Code Snippet counts)
       - Source reputation (consider libraries with High or Medium reputation more authoritative)
       - Benchmark score (higher is better, 100 is the maximum)
    3. If multiple good matches exist, acknowledge this but proceed with the most relevant one
    4. If no good matches exist, clearly state this and suggest query refinements
    5. For ambiguous queries, request clarification before proceeding with a best-guess match
    
    ### Version-specific IDs
    
    If the user mentions a specific version, use a version-specific library ID:
    
    ```bash
    # General (latest indexed)
    npx ctx7@latest docs /vercel/next.js "How to set up app router"
    
    # Version-specific
    npx ctx7@latest docs /vercel/next.js/v14.3.0-canary.87 "How to set up app router"
    ```
    
    The available versions are listed in the `library` command output. Use the closest match to what the user specified.
    
    ## Step 2: Query Documentation
    
    Retrieves up-to-date documentation and code examples for the resolved library.
    
    ```bash
    npx ctx7@latest docs /facebook/react "How to clean up useEffect with async operations"
    npx ctx7@latest docs /vercel/next.js "How to add authentication middleware to app router"
    npx ctx7@latest docs /prisma/prisma "How to define one-to-many relations with cascade delete"
    ```
    
    ### Writing good queries
    
    The query directly affects the quality of results. Be specific and include relevant details, but keep each query to one topic — if the question spans multiple distinct concepts, run a separate `docs` command per concept instead of combining them, unless the question is about how the concepts interact. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.
    
    | Quality | Example |
    |---------|---------|
    | Good | `"How to set up authentication with JWT in Express.js"` |
    | Good | `"React useEffect cleanup function with async operations"` |
    | Bad (too vague) | `"auth"` |
    | Bad (too vague) | `"hooks"` |
    | Bad (too broad) | `"routing and auth and caching in Next.js"` |
    
    Describe what to look up in the library's documentation, rather than the task to complete — vague one-word queries return generic results, and multi-topic queries dilute ranking and return shallow results for each topic.
    
    The output contains two types of content: **code snippets** (titled, with language-tagged blocks) and **info snippets** (prose explanations with breadcrumb context).
    
    ## Authentication
    
    Works without authentication. For higher rate limits:
    
    ```bash
    # Option A: environment variable
    export CONTEXT7_API_KEY=your_key
    
    # Option B: OAuth login
    npx ctx7@latest login
    ```
    
    ## Error Handling
    
    If a command fails with a quota error ("Monthly quota reached" or "quota exceeded"):
    1. Inform the user their Context7 quota is exhausted
    2. Suggest they authenticate for higher limits: `npx ctx7@latest login`
    3. If they cannot or choose not to authenticate, answer from training knowledge and clearly note it may be outdated
    
    Do not silently fall back to training data — always tell the user why Context7 was not used.
    
    ## Common Mistakes
    
    - Library IDs require a `/` prefix — `/facebook/react` not `facebook/react`
    - Always run `npx ctx7@latest library` first — `npx ctx7@latest docs react "hooks"` will fail without a valid ID
    - Use descriptive queries, not single words — `"React useEffect cleanup function"` not `"hooks"`
    - One topic per query — split `"routing and auth and caching"` into a separate `docs` command per concept, unless the question is about how they interact
    - Do not include sensitive information (API keys, passwords, credentials) in queries
    ✓ • 0ms

## Activity

    $ cat SKILL.md references/verification.md; sed -n '1,260p' scripts/maintenance_state.py
    ---
    name: project-maintenance
    description: Keep a project's code maps, implementations, tests and build routes current after feature work, refactoring, API changes or backend migrations. Use for maintenance and retirement of superseded code in assembly, Rust, Python, TypeScript, C and C++ projects, especially native UI and dedicated inference engines.
    ---
    
    # Project Maintenance
    
    把维护作为当前开发任务的完成条件。同步受影响的地图，清除已被替代的实现和失效测试，验证仍然承诺的产品行为。对用户已经授权的任务内维护直接执行；跨出任务范围、改变支持平台或对外接口时，先给出具体影响。
    
    ## 确认当前契约
    
    - 从指定项目根目录开始，读取 `AGENTS.md`、当前需求及已有地图。保留规定的语言、渲染后端、推理引擎、平台支持和源码所有权边界。
    - 检查已提交、暂存、未暂存及新增文件。区分本次任务和其他人的未完成工作，不替别人整理不相关改动。
    - 新项目随首个可运行实现建立地图；已有项目以当前源码验证地图，不以旧文档推断不存在的接口。
    - 链接进 UI 仓库的引擎源码、子模块和独立 checkout 各自验证真实目录；一个仓库的记录不证明另一个源码树已经同步。
    
    Git 项目可先运行只读检查：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    ```
    
    脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    
    ## 随改动更新地图
    
    ### 双射性与增删改
    
    每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
    
    - **增**：补上当前契约所需的实现、有效验证和遗漏的地图条目。
    - **删**：删除已确认冗余或退役的代码，以及其专属测试、依赖、构建路线、文档和地图条目。
    - **改**：优先原地修正仍承担职责的实现，同步职责、接口、依赖、路径和行范围；不靠另加平行版本绕过旧问题。
    
    双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”。按 `$codemap` 格式，每个文件在本层 Files 中恰好一条，每个直接子目录在 Subdirectories 中恰好一条；其他格式明确等价的归属。纯重导出文件也不能因导出表存在而漏记。任务入口、符号表和依赖边允许多处指向同一实体，不构成重复归属；不要求每个函数或使用场景各有一张表。索引自身不递归作为源码索引。
    
    以当前工作树双向核对：源码到地图查缺漏和重复归属，地图到源码查幽灵路径和失真的职责、接口、依赖及行范围。明确排除范围，不能通过扩大忽略规则掩盖缺漏。完成时简要说明增删改的处理或不适用依据，以及仍未闭合的缺口。
    
    双射只约束声明的索引范围。根据用户关注点和项目约定，不关心的部分可以明确忽略或只保留目录边界，无需逐文件维护。必须在根地图的“范围与忽略项”中记录 `根目录相对路径/模式 | 处理方式 | 原因/依据`；处理方式为“排除”或“仅保留边界”。后者保留唯一目录条目并标注“内部未索引”，不要求内部文件表或子地图。常规忽略项可按默认规则、`.gitignore` 等明确来源归组，项目特有忽略项及重新纳入的例外必须明确列出。根地图是范围的权威记录，其他位置只引用。
    
    忽略索引不等于过时、可删除或免于正确性验证。后续任务触及忽略区或依赖其契约时，读取所需代码并重新评估范围；决定变化时同步标记，不能把未索引内容宣称为已核对。维护脚本的扫描排除与地图的索引排除分别服务于变更检测和导航，不能未经判断直接互相套用。
    
    使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    
    - 文件增删、移动、重命名：同步本层文件表、父目录路由及任务入口，移除已不存在目录的地图。
    - 接口或依赖变化：同步导出符号、调用入口、跨目录依赖及消费者。
    - 汇编与 FFI：记录真实符号、调用约定、结构布局和偏移、内存所有权、回调线程，以及 C/Rust 包装层；用头文件、声明、调用点和链接符号核对。
    - UI 与推理引擎：记录事件到状态到渲染/推理的路径；模型边界写明 tensor shape、dtype、布局、量化格式和必要的转换。
    - 实现变化：若职责、数据流、契约或大文件功能行范围变化，仍然更新相关说明；纯内部等价修改可以不改地图，并说明判断依据。
    
    地图保留一份当前版本，原地更新。只在 agent 规则文件中放简短入口和完成条件，避免复制整份地图。
    
    ## 文档一起更新和退役
    
    - 代码、接口、命令、目录、配置、支持平台或用户流程变化时，检查 README、架构说明、运行手册、示例、任务入口和交叉链接；把仍然有效的说明改成当前行为。
    - 功能、接口或构建路线退役时，删除只服务于它的文档、示例、截图、fixture、导航条目和失效链接；先核对反向链接与仍在使用的共享资源。Git 历史承担已删除内容的留存。
    - 保留一份权威说明，合并重复文档，避免用过时的 `old/`、`v1/`、`backup/` 文档树掩盖当前状态。文档删除必须有当前契约或调用关系依据，不能只因文件很旧或无人引用。
    - 完成维护后，至少检查受影响 Markdown 链接、命令、路径和状态描述；不能验证的外部链接或平台行为要明确记录。
    
    ## 实现与测试一起退役
    
    **时效淘汰性**：每次变更主动复查受影响旧实现和临时层的保留依据、迁移状态与移除条件。替代实现已验证、消费者已迁移或旧需求已撤销时，在本次任务内完成退役；不能仅在地图上标成 deprecated 后无限保留。暂时仍需保留的分支必须记录具体消费者或当前契约及可检查的移除条件，在后续相关改动时重新核对。时效由当前需求和迁移证据判定，不按文件年龄或任意期限机械删除。
    
    对候选项先核对用途、调用者、构建目标、平台和公开契约，再在本次授权范围内处理：
    
    - 已替换并完成迁移的实现：清除旧代码、专属包装层、构建入口、依赖、文档、测试与 fixture；历史由 Git 保留，不复制 `old/`、`v1/`、`backup/` 源码树。
    - 临时适配层：迁移完成就移除；仍有使用者时，写清当前消费者和具体移除条件。
    - 重复或仅验证 mock 自身的测试：合并或删除，保留不同输入、平台、边界条件和已发现故障的有效断言。
    - 测试取消或改变的需求：按当前契约退役或重写，同时核对相关 fixture、runner 配置和文档。
    - 失败、被跳过、年代久远、覆盖率低或图中没有入边，都不是单独的删除依据。先区分产品回归、环境问题、有效平台测试和真正失效的需求。
    - 新测试只验证本次行为、接口或实际故障，避免把当前实现细节再写一遍。对仍有效的回归保持独立检测能力；不能为使检查通过而删除断言、改宽误差或重录错误的快照。
    
    外部导出、动态注册、函数指针、FFI、汇编入口和跨平台分支需要额外核对消费者。只证明“无本地文本引用”时，报告剩余不确定性，继续完成其余可确认的维护。
    
    ## 验证并记录当前状态
    
    按 [验证与退役判据](references/verification.md) 只选择本次涉及的语言、平台、UI 或推理验证。优先复用现有验证入口；不要为了使用这个 skill 重建一套重复测试框架。
    
    完成本次维护和有效验证后，在原地替换单份审查记录：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py record --root <project-root> --reviewed
    python3 <skill-dir>/scripts/maintenance_state.py check --root <project-root>
    ```
    
    `--reviewed` 表示 agent 已实际审查当前范围并完成应做的验证，不是用户批准，也不能替代验证。未完成的关联改动、未知失败或未验证的其他源码树必须如实报告；不能为让检查变绿而重录。
    
    记录位于 `.project-maintenance/state.json`，包含当前文件哈希与审查版本。`check` 在文件或审查范围变化后返回非零，供已有验证命令、pre-commit 或 CI 调用；新增这些集成时使用项目现有方式，编辑一个可重复更新的入口。脚本的范围、排除模式和能力限制见验证参考。
    
    最终报告本次同步的地图、退役项及其依据、实际验证结果和未解决问题。指标是当前契约和可维护性，不是删除数量、测试数量或通过率。
    # 验证与退役判据
    
    只读取本次涉及的部分。使用项目已有命令、锁定版本、功能组合和支持平台，先记录已存在的失败，再核对本次差异。缺少完成验证所必需的工具或数据时补齐并执行；不能把未验证写成通过。
    
    ## 语言与接口边界
    
    | 场景 | 需要的证据 |
    | --- | --- |
    | 汇编、C、C++ | 用实际目标和编译选项构建并链接；核对导出/引用符号、ABI、栈与缓冲区对齐、寄存器保存、布局断言及有效的平台分支。删汇编符号前核对 C/Rust 声明、链接脚本、动态查找和函数指针注册。 |
    | Rust / FFI | 当前 feature 与 target 下的构建和接口测试；对应安全契约、布局、所有权及异常边界。Miri 可验证 Rust 侧契约，但不能证明任意 C/汇编实际调用正确。 |
    | Python | 当前环境的类型、lint 和相关功能验证；核对动态导入、插件注册、CLI、模型导出/转换入口及包的公开接口。 |
    | TypeScript | 当前配置的类型检查、构建与相关行为测试；核对动态加载、路由、注册表、包导出及原生桥接。死代码工具只提供候选。 |
    
    ## 原生与 Web UI
    
    - 原生 UI：运行实际窗口，验证此次涉及的输入、焦点、文本、缩放、布局、渲染路径及与真实引擎的交互；检查截图/帧与运行日志。构建成功和生成一张图不证明交互完整。
    - Web UI：在真实浏览器中验证此次流程、控制台、网络和相应视口；使用当前页面状态定位元素。纯 DOM 或组件快照不证明连接实际后端后的流程。
    - 视觉基线只在需求确认改变且实际画面核对后更新。移除弃用界面的快照与 fixture 时，同时核对共用资源的消费者。
    
    ## 专用推理框架
    
    - 从原始模型或已验证的参考实现建立输入与输出契约：shape、dtype、stride/layout、量化参数、采样/帧约定以及适用精度。
    - 按实际工作负载检查数值对齐和完整推理路径。新内核需要验证尾部尺寸、对齐、有效 ISA 分派和 fallback；“更快”不证明“相同”。
    - 性能采用同机、同负载、同契约的稳定比较，记录端到端与真实热点。复用已有 `performance-gradient-optimization` 或项目等效方法。
    - 保留当前对齐/性能基线的明确用途；淘汰无用的候选实现、重复基准入口和失效数据。基线输入、期望结果与硬件/模型来源需要可追溯，历史结果交由项目既有存档机制保存。
    
    ## 持续维护检查脚本
    
    `scripts/maintenance_state.py` 仅依赖 Python 标准库和 Git，支持包括尚无首次提交的 Git 工作树：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py inspect --root . --json
    python3 <skill-dir>/scripts/maintenance_state.py record --root . --reviewed
    python3 <skill-dir>/scripts/maintenance_state.py check --root .
    ```
    
    - 扫描 Git 跟踪文件及未被忽略的新增文件，比较实际内容哈希；检测修改、增删、重命名及地图本身的变化。模式或时间戳相同的内容修改仍会触发。
    - tracked fixture、lockfile、配置与文档也在范围内。`CODEMAP.md`、`codemap.md` 和 `*.analysis.md` 在报告中单列；记录文件自身不进入指纹。
    - 默认排除缓存/依赖目录，以及根目录的 `build/`、`dist/`、`out/`、`coverage/`。若项目把真实源码放在这些位置，通过 `--include-generated` 纳入；该选择会记录在范围中。
    - 大模型、数据集或受项目明确排除的产物，可使用根目录相对的 `--exclude 'weights/**'` 等模式；目录模式如 `datasets/` 排除其子树。模式影响扫描范围，不能拿它掩盖未完成维护。设置会保存在审查记录中，后续检查复用；显式重新提供模式会替换已保存的自定义排除。
    - 例如：`inspect --root . --exclude 'weights/**'`；确认范围后以相同选项运行 `record --reviewed`。恢复自定义排除为空使用 `--clear-excludes`。修改排除范围会要求新的审查记录。
    - 符号链接只记录链接目标，不跟随读取外部文件；子模块只记录 checkout 状态。报告中的 `separate_checkouts` 必须在其真实源码树单独审查和检查，链接本身不变不代表引擎内容不变。
    - `record` 只原子替换 `.project-maintenance/state.json`。若使用 CI，可把这份记录交给 Git；后续在同一路径更新，不生成每次任务的副本。
    - `inspect` 只读并返回清单。`check` 返回 `0` 表示该范围与已审查指纹一致，`1` 表示需审查或尚无记录，`2` 表示仓库、读取或记录错误；错误不能当作干净状态。
    - 一致的指纹只证明“之后没有新的受监测文件变化”，不证明地图语义、死代码判断或产品正确性。实际调用者和有意义的验证仍由 agent 核对。
    
    在已有项目中接入检查脚本时使用该 skill 的实际路径或项目自己的固定入口。不要假定其他机器安装在同一个 home 路径，也不要把 skill 私有源码大量复制进项目。
    #!/usr/bin/env python3
    """Track changes since a deliberate project-maintenance review; never delete code."""
    
    from __future__ import annotations
    
    import argparse
    from datetime import datetime, timezone
    import fnmatch
    import hashlib
    import json
    import os
    from pathlib import Path
    import subprocess
    import sys
    import tempfile
    
    
    STATE_PATH = Path(".project-maintenance/state.json")
    SCHEMA_VERSION = 1
    CACHE_DIRS = {
        ".git", ".hg", ".svn", "node_modules", "target", ".venv", "venv",
        "__pycache__", ".pytest_cache", ".mypy_cache", ".ruff_cache", ".cache",
    }
    GENERATED_ROOTS = {"build", "dist", "out", "coverage"}
    
    
    def git(root: Path, *args: str, optional: bool = False) -> bytes:
        result = subprocess.run(
            ["git", "-C", str(root), *args], capture_output=True, check=False
        )
        if result.returncode and not optional:
            raise ValueError(result.stderr.decode(errors="replace").strip())
        return b"" if result.returncode else result.stdout
    
    
    def repo_root(directory: str) -> Path:
        result = git(Path(directory).resolve(), "rev-parse", "--show-toplevel")
        return Path(os.fsdecode(result.rstrip(b"\n"))).resolve()
    
    
    def load_state(root: Path) -> dict | None:
        path = root / STATE_PATH
        if not path.exists():
            return None
        if path.is_symlink():
            raise ValueError("Maintenance state must be a regular file, not a symlink")
        state = json.loads(path.read_text(encoding="utf-8"))
        if not isinstance(state, dict) or state.get("schema_version") != SCHEMA_VERSION:
            raise ValueError("Unsupported or malformed maintenance state")
        for field in ("files", "maps"):
            entries = state.get(field)
            if not isinstance(entries, dict) or not all(
                isinstance(k, str) and isinstance(v, str) for k, v in entries.items()
            ):
                raise ValueError(f"Malformed state field: {field}")
        if not isinstance(state.get("excludes"), list) or not all(
            isinstance(p, str) for p in state["excludes"]
        ) or not isinstance(state.get("include_generated"), bool):
            raise ValueError("Malformed maintenance scope")
        return state
    
    
    def excluded(name: str, patterns: list[str], include_generated: bool) -> bool:
        parts = Path(name).parts
        if not parts or parts[0] == STATE_PATH.parts[0]:
            return True
        if any(part in CACHE_DIRS for part in parts[:-1]):
            return True
        if not include_generated and parts[0] in GENERATED_ROOTS:
            return True
        return any(
            name.startswith(pattern) if pattern.endswith("/")
            else fnmatch.fnmatchcase(name, pattern)
            for pattern in patterns
        )
    
    
    def fingerprint(path: Path) -> tuple[str, dict | None]:
        if path.is_symlink():
            target = os.readlink(path)
            digest = hashlib.sha256(os.fsencode(target)).hexdigest()
            return "symlink:" + digest, {"kind": "symlink", "target": target}
        if path.is_dir():
            # Git ls-files emits a directory only for a gitlink. Its own tree needs review.
            child_root = git(path, "rev-parse", "--show-toplevel", optional=True)
            initialized = bool(child_root) and Path(
                os.fsdecode(child_root.rstrip(b"\n"))
            ).resolve() == path.resolve()
            if initialized:
                payload = git(path, "rev-parse", "HEAD", optional=True) + git(
                    path, "status", "--porcelain=v1", "-z", "--untracked-files=all"
                )
            else:
                payload = b"uninitialized"
            return "gitlink:" + hashlib.sha256(payload).hexdigest(), {
                "kind": "submodule", "initialized": initialized,
            }
        before = path.stat()
        digest = hashlib.sha256()
        with path.open("rb") as stream:
            for block in iter(lambda: stream.read(1024 * 1024), b""):
                digest.update(block)
        after = path.stat()
        if (before.st_size, before.st_mtime_ns, before.st_mode) != (
            after.st_size, after.st_mtime_ns, after.st_mode
        ):
            raise ValueError(f"File changed during scan; retry: {path}")
        executable = "x" if before.st_mode & 0o111 else "-"
        return "file:" + executable + ":" + digest.hexdigest(), None
    
    
    def inventory(root: Path, patterns: list[str], include_generated: bool) -> dict:
        output = git(root, "ls-files", "-z", "--cached", "--others", "--exclude-standard")
        names = sorted({os.fsdecode(name) for name in output.split(b"\0") if name})
        files, maps, separate = {}, {}, []
        for name in names:
            if excluded(name, patterns, include_generated):
                continue
            path = root / name
            if not os.path.lexists(path):
                continue  # Tracked deletions are absent from the current content manifest.
            value, checkout = fingerprint(path)
            target = maps if path.name in {"CODEMAP.md", "codemap.md"} or (
                path.name.endswith(".analysis.md")
            ) else files
            target[name] = value
            if checkout:
                separate.append({"path": name, **checkout})
        return {"files": files, "maps": maps, "separate_checkouts": separate}
    
    
    def differences(old: dict, new: dict) -> dict:
        return {
            "added": sorted(new.keys() - old.keys()),
            "modified": sorted(k for k in old.keys() & new.keys() if old[k] != new[k]),
            "removed": sorted(old.keys() - new.keys()),
        }
    
    
    def write_state(root: Path, state: dict) -> None:
        directory = root / STATE_PATH.parent
        if directory.is_symlink():
            raise ValueError("Maintenance state directory must not be a symlink")
        directory.mkdir(exist_ok=True)
        temporary = None
        try:
            with tempfile.NamedTemporaryFile(
                mode="w", encoding="utf-8", dir=directory, prefix=".state-", delete=False
            ) as stream:
                temporary = Path(stream.name)
                json.dump(state, stream, indent=2, sort_keys=True)
                stream.write("\n")
            os.replace(temporary, directory / STATE_PATH.name)
        finally:
            if temporary is not None and temporary.exists():
                temporary.unlink()
    
    
    def print_report(report: dict, as_json: bool) -> None:
        if as_json:
            print(json.dumps(report, indent=2, sort_keys=True))
            return
        print(f"Project: {report['root']}")
        print(f"Status: {report['status']}")
        for group in ("files", "maps"):
            print(f"{group}: {report['counts'][group]} monitored")
            for kind, paths in report["changes"][group].items():
                if paths:
                    print(f"  {kind}: {len(paths)}")
                    for name in paths[:40]:
                        print("    " + ascii(name))
                    if len(paths) > 40:
                        print("    ... use --json for the complete list")
        if report["scope_changed"]:
            print("Scope changed; review exclusions before recording")
        if report["separate_checkouts"]:
            print("Linked/submodule content requires separate review:")
            for checkout in report["separate_checkouts"]:
                print("  " + ascii(checkout["path"]) + " (" + checkout["kind"] + ")")
    
    
    def main() -> int:
        parser = argparse.ArgumentParser(description=__doc__)
        parser.add_argument("action", choices=("inspect", "record", "check"))
        parser.add_argument("--root", required=True, help="Path inside the intended Git project")
        parser.add_argument("--json", action="store_true", help="Emit complete structured output")
        parser.add_argument("--reviewed", action="store_true", help="Record only after actual review")
        parser.add_argument("--exclude", action="append", help="Replace custom root-relative exclusions")
        parser.add_argument("--clear-excludes", action="store_true")
        parser.add_argument("--include-generated", action="store_true", default=None)
        parser.add_argument("--exclude-generated", dest="include_generated", action="store_false", default=None)
        args = parser.parse_args()
        if args.action == "record" and not args.reviewed:
            parser.error("record requires --reviewed after maintenance and verification")
        if args.reviewed and args.action != "record":
            parser.error("--reviewed applies only to record")
        if args.exclude is not None and args.clear_excludes:
            parser.error("Use --exclude or --clear-excludes, not both")
        try:
            root = repo_root(args.root)
            old = load_state(root)
            patterns = sorted(set(args.exclude)) if args.exclude is not None else (
                [] if args.clear_excludes else (old or {}).get("excludes", [])
            )
            if any(not p or p.startswith("/") or ".." in Path(p).parts for p in patterns):
                raise ValueError("Exclusions must be nonempty root-relative patterns without '..'")
            include_generated = args.include_generated if args.include_generated is not None else (
                (old or {}).get("include_generated", False)
            )
            current = inventory(root, patterns, include_generated)
            changes = {group: differences((old or {}).get(group, {}), current[group])
                       for group in ("files", "maps")}
            scope_changed = old is not None and (
                patterns != old["excludes"] or include_generated != old["include_generated"]
            )
            needs_review = old is None or scope_changed or any(
                paths for group in changes.values() for paths in group.values()
            )
            status = "review-required" if needs_review else "matches-reviewed-state"
            if args.action == "record":
                state = {
                    "schema_version": SCHEMA_VERSION,
                    "reviewed_at": datetime.now(timezone.utc).isoformat(),
                    "head": git(root, "rev-parse", "--verify", "HEAD", optional=True).decode().strip() or None,
                    "excludes": patterns, "include_generated": include_generated, **current,
                }
                write_state(root, state)
                status = "recorded-reviewed-state"
            print_report({
                "root": str(root), "status": status, "scope_changed": scope_changed,
                "counts": {group: len(current[group]) for group in ("files", "maps")},
                "changes": changes, "separate_checkouts": current["separate_checkouts"],
                "excludes": patterns, "include_generated": include_generated,
            }, args.json)
            return int(needs_review) if args.action == "check" else 0
        except (OSError, ValueError, UnicodeError) as error:
            if args.json:
                print(json.dumps({"error": str(error)}), file=sys.stderr)
            else:
                print(f"Error: {error}", file=sys.stderr)
            return 2
    
    
    if __name__ == "__main__":
        sys.exit(main())
    ✓ • 0ms

## Activity

    $ git rev-parse --show-toplevel; git diff -- SKILL.md; git diff --stat; cat agents/openai.yaml; cat /home/fuurin/.codex/skills/find-docs/SKILL.md
    /home/fuurin/code/skills
    diff --git a/project-maintenance/SKILL.md b/project-maintenance/SKILL.md
    index 8cc02dc..e233fd8 100644
    --- a/project-maintenance/SKILL.md
    +++ b/project-maintenance/SKILL.md
    @@ -24,6 +24,22 @@ python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    
     ## 随改动更新地图
    
    +### 双射性与增删改
    +
    +每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
    +
    +- **增**：补上当前契约所需的实现、有效验证和遗漏的地图条目。
    +- **删**：删除已确认冗余或退役的代码，以及其专属测试、依赖、构建路线、文档和地图条目。
    +- **改**：优先原地修正仍承担职责的实现，同步职责、接口、依赖、路径和行范围；不靠另加平行版本绕过旧问题。
    +
    +双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”。按 `$codemap` 格式，每个文件在本层 Files 中恰好一条，每个直接子目录在 Subdirectories 中恰好一条；其他格式明确等价的归属。纯重导出文件也不能因导出表存在而漏记。任务入口、符号表和依赖边允许多处指向同一实体，不构成重复归属；不要求每个函数或使用场景各有一张表。索引自身不递归作为源码索引。
    +
    +以当前工作树双向核对：源码到地图查缺漏和重复归属，地图到源码查幽灵路径和失真的职责、接口、依赖及行范围。明确排除范围，不能通过扩大忽略规则掩盖缺漏。完成时简要说明增删改的处理或不适用依据，以及仍未闭合的缺口。
    +
    +双射只约束声明的索引范围。根据用户关注点和项目约定，不关心的部分可以明确忽略或只保留目录边界，无需逐文件维护。必须在根地图的“范围与忽略项”中记录 `根目录相对路径/模式 | 处理方式 | 原因/依据`；处理方式为“排除”或“仅保留边界”。后者保留唯一目录条目并标注“内部未索引”，不要求内部文件表或子地图。常规忽略项可按默认规则、`.gitignore` 等明确来源归组，项目特有忽略项及重新纳入的例外必须明确列出。根地图是范围的权威记录，其他位置只引用。
    +
    +忽略索引不等于过时、可删除或免于正确性验证。后续任务触及忽略区或依赖其契约时，读取所需代码并重新评估范围；决定变化时同步标记，不能把未索引内容宣称为已核对。维护脚本的扫描排除与地图的索引排除分别服务于变更检测和导航，不能未经判断直接互相套用。
    +
     使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    
     - 文件增删、移动、重命名：同步本层文件表、父目录路由及任务入口，移除已不存在目录的地图。
    @@ -43,6 +59,8 @@ python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    
     ## 实现与测试一起退役
    
    +**时效淘汰性**：每次变更主动复查受影响旧实现和临时层的保留依据、迁移状态与移除条件。替代实现已验证、消费者已迁移或旧需求已撤销时，在本次任务内完成退役；不能仅在地图上标成 deprecated 后无限保留。暂时仍需保留的分支必须记录具体消费者或当前契约及可检查的移除条件，在后续相关改动时重新核对。时效由当前需求和迁移证据判定，不按文件年龄或任意期限机械删除。
    +
     对候选项先核对用途、调用者、构建目标、平台和公开契约，再在本次授权范围内处理：
    
     - 已替换并完成迁移的实现：清除旧代码、专属包装层、构建入口、依赖、文档、测试与 fixture；历史由 Git 保留，不复制 `old/`、`v1/`、`backup/` 源码树。
     codemap/SKILL.md             | 38 ++++++++++++++++++++++++++++++++------
     project-maintenance/SKILL.md | 18 ++++++++++++++++++
     2 files changed, 50 insertions(+), 6 deletions(-)
    interface:
      display_name: "Project Maintenance"
      short_description: "Keep code maps, active code and tests current"
      default_prompt: "Use $project-maintenance to finish this change by updating code maps, retiring obsolete code and tests, and verifying the current product."
    ---
    name: find-docs
    description: >-
      Retrieves up-to-date documentation, API references, and code examples for any
      developer technology. Use this skill whenever the user asks about a specific
      library, framework, SDK, CLI tool, or cloud service — even for well-known ones
      like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. Your
      training data may not reflect recent API changes or version updates.
    
      Always use for: API syntax questions, configuration options, version migration
      issues, "how do I" questions mentioning a library name, debugging that involves
      library-specific behavior, setup instructions, and CLI tool usage.
    
      Use even when you think you know the answer — do not rely on training data
      for API details, signatures, or configuration options as they are frequently
      outdated. Always verify against current docs. Prefer this over web search for
      library documentation and API details.
    ---
    
    # Documentation Lookup
    
    Retrieve current documentation and code examples for any library using the Context7 CLI.
    
    Run commands with `npx ctx7@latest` so setup always uses the latest CLI without a global install:
    
    ```bash
    npx ctx7@latest library <name> "<query>"
    npx ctx7@latest docs <libraryId> "<query>"
    ```
    
    Optionally install globally if you prefer a bare `ctx7` command:
    
    ```bash
    npm install -g ctx7@latest
    ```
    
    ## Workflow
    
    Two-step process: resolve the library name to an ID, then query docs with that ID.
    
    ```bash
    # Step 1: Resolve library ID
    npx ctx7@latest library <name> "<query>"
    
    # Step 2: Query documentation
    npx ctx7@latest docs <libraryId> "<query>"
    ```
    
    You MUST call `library` first to obtain a valid library ID UNLESS the user explicitly provides a library ID in the format `/org/project` or `/org/project/version`.
    
    IMPORTANT: Do not run these commands more than 3 times per question. If you cannot find what you need after 3 attempts, use the best result you have.
    
    ## Step 1: Resolve a Library
    
    Resolves a package/product name to a Context7-compatible library ID and returns matching libraries.
    
    ```bash
    npx ctx7@latest library React "How to clean up useEffect with async operations"
    npx ctx7@latest library "Next.js" "How to set up app router with middleware"
    npx ctx7@latest library Prisma "How to define one-to-many relations with cascade delete"
    ```
    
    Use the official library name with proper punctuation (e.g., "Next.js" not "nextjs", "Customer.io" not "customerio", "Three.js" not "threejs"). If results look wrong, try alternate spellings such as `next.js` before changing the query.
    
    Always pass a `query` argument — it is required and directly affects result ranking. Use the user's intent to form the query, which helps disambiguate when multiple libraries share a similar name. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.
    
    ### Result fields
    
    Each result includes:
    
    - **Library ID** — Context7-compatible identifier (format: `/org/project`)
    - **Name** — Library or package name
    - **Description** — Short summary
    - **Code Snippets** — Number of available code examples
    - **Source Reputation** — Authority indicator (High, Medium, Low, or Unknown)
    - **Benchmark Score** — Quality indicator (100 is the highest score)
    - **Versions** — List of versions if available. Use one of those versions if the user provides a version in their query. The format is `/org/project/version`.
    
    ### Selection process
    
    1. Analyze the query to understand what library/package the user is looking for
    2. Select the most relevant match based on:
       - Name similarity to the query (exact matches prioritized)
       - Description relevance to the query's intent
       - Documentation coverage (prioritize libraries with higher Code Snippet counts)
       - Source reputation (consider libraries with High or Medium reputation more authoritative)
       - Benchmark score (higher is better, 100 is the maximum)
    3. If multiple good matches exist, acknowledge this but proceed with the most relevant one
    4. If no good matches exist, clearly state this and suggest query refinements
    5. For ambiguous queries, request clarification before proceeding with a best-guess match
    
    ### Version-specific IDs
    
    If the user mentions a specific version, use a version-specific library ID:
    
    ```bash
    # General (latest indexed)
    npx ctx7@latest docs /vercel/next.js "How to set up app router"
    
    # Version-specific
    npx ctx7@latest docs /vercel/next.js/v14.3.0-canary.87 "How to set up app router"
    ```
    
    The available versions are listed in the `library` command output. Use the closest match to what the user specified.
    
    ## Step 2: Query Documentation
    
    Retrieves up-to-date documentation and code examples for the resolved library.
    
    ```bash
    npx ctx7@latest docs /facebook/react "How to clean up useEffect with async operations"
    npx ctx7@latest docs /vercel/next.js "How to add authentication middleware to app router"
    npx ctx7@latest docs /prisma/prisma "How to define one-to-many relations with cascade delete"
    ```
    
    ### Writing good queries
    
    The query directly affects the quality of results. Be specific and include relevant details, but keep each query to one topic — if the question spans multiple distinct concepts, run a separate `docs` command per concept instead of combining them, unless the question is about how the concepts interact. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.
    
    | Quality | Example |
    |---------|---------|
    | Good | `"How to set up authentication with JWT in Express.js"` |
    | Good | `"React useEffect cleanup function with async operations"` |
    | Bad (too vague) | `"auth"` |
    | Bad (too vague) | `"hooks"` |
    | Bad (too broad) | `"routing and auth and caching in Next.js"` |
    
    Describe what to look up in the library's documentation, rather than the task to complete — vague one-word queries return generic results, and multi-topic queries dilute ranking and return shallow results for each topic.
    
    The output contains two types of content: **code snippets** (titled, with language-tagged blocks) and **info snippets** (prose explanations with breadcrumb context).
    
    ## Authentication
    
    Works without authentication. For higher rate limits:
    
    ```bash
    # Option A: environment variable
    export CONTEXT7_API_KEY=your_key
    
    # Option B: OAuth login
    npx ctx7@latest login
    ```
    
    ## Error Handling
    
    If a command fails with a quota error ("Monthly quota reached" or "quota exceeded"):
    1. Inform the user their Context7 quota is exhausted
    2. Suggest they authenticate for higher limits: `npx ctx7@latest login`
    3. If they cannot or choose not to authenticate, answer from training knowledge and clearly note it may be outdated
    
    Do not silently fall back to training data — always tell the user why Context7 was not used.
    
    ## Common Mistakes
    
    - Library IDs require a `/` prefix — `/facebook/react` not `facebook/react`
    - Always run `npx ctx7@latest library` first — `npx ctx7@latest docs react "hooks"` will fail without a valid ID
    - Use descriptive queries, not single words — `"React useEffect cleanup function"` not `"hooks"`
    - One topic per query — split `"routing and auth and caching"` into a separate `docs` command per concept, unless the question is about how they interact
    - Do not include sensitive information (API keys, passwords, credentials) in queries
    ✓ • 0ms

## Activity

    $ for tool in rg git python3 jq sqlite3 ast-grep sg ctags clangd rust-analyzer pyright; do command -v "$tool" || true; done; rg --files ../codemap /home/fuurin/.codex/skills/.system/skill-creator/scripts; sed -n '1,230p' ../codemap/SKILL.md
    /home/fuurin/.codex/packages/app-server-daemon/releases/0.161.0-x86_64-unknown-linux-musl/codex-path/rg
    /usr/sbin/git
    /usr/sbin/python3
    /usr/sbin/jq
    /usr/sbin/sqlite3
    /usr/sbin/clangd
    /home/fuurin/.cargo/bin/rust-analyzer
    /home/fuurin/.local/bin/pyright
    /home/fuurin/.codex/skills/.system/skill-creator/scripts/init_skill.py
    /home/fuurin/.codex/skills/.system/skill-creator/scripts/generate_openai_yaml.py
    /home/fuurin/.codex/skills/.system/skill-creator/scripts/quick_validate.py
    /home/fuurin/.codex/skills/.system/skill-creator/scripts/__pycache__/generate_openai_yaml.cpython-311.pyc
    ../codemap/LICENSE
    ../codemap/README.md
    ../codemap/README_EN.md
    ../codemap/SKILL.md
    ---
    name: codemap
    description: Use when indexing a codebase for agent navigation, generating or refreshing CODEMAP.md files, mapping large project structure, or adding CODEMAP guidance for learning/maintenance workflows.
    ---
    
    # CODEMAP — Codebase Navigation Index Generator
    
    Generate hierarchical `CODEMAP.md` files that help agents locate relevant code without scanning unrelated files. Core navigation: **Task Guide first → Domain filter → targeted reads**. For files over 1000 lines, generate a companion `<filename>.analysis.md` with intent-to-line-range mapping.
    
    ## Core Principles
    
    - `CODEMAP.md` is a navigation constraint, not documentation to browse.
    - Prefer positive guidance (Task Guide, Domain, Key Exports) over broad listings.
    - Dependencies are a safety net, not an invitation to chain-read.
    - Each CODEMAP describes only its own directory level. Child directory details belong in child CODEMAPs.
    - Maintain a bijection between in-scope source entities and canonical inventory entries: no omissions, no duplicate ownership, no entries for nonexistent entities. Navigation references may be many-to-one; they are not additional inventory entries.
    - Every code change requires considering addition, deletion, and modification together. Maintenance also actively retires superseded code within the authorized task scope; adding an index entry does not complete a migration.
    
    ## Bijection and Freshness
    
    Define the inventory scope using the declared ignore rules. Every included immediate source file has exactly one canonical row in its directory's Files table; every included immediate child directory has exactly one Subdirectories row. Parent maps route to children without duplicating their inventories. CODEMAPs and companion analysis files are index artifacts, not source entities to recursively index. Key Exports and Task Guide are selective navigation views, not exhaustive inventories of all symbols or tasks.
    
    Validate both directions against the current source tree, including uncommitted changes: source → map catches omissions and duplicate ownership; map → source catches stale paths, symbols, responsibilities, dependencies, and line ranges. Do not exclude a source merely to hide an omission. Shared implementations can have multiple callers and navigation references without duplicate canonical entries.
    
    Freshness is a completion condition after each code change, not a timestamp update. In maintenance mode, inspect affected implementations for replacement, duplication, obsolete requirements, and completed migration scaffolding. Retire confirmed obsolete code and its exclusive tests, dependencies, build routes, documentation, and map entries in the same task. Retained compatibility paths need concrete consumers or a current contract and an explicit removal condition; recheck that condition when affected. Age or absence of local text references alone is not proof of obsolescence. Learning mode remains read-only for source code.
    
    ## Language Rule
    
    Generated files use the user's request language for prose/headings. Code identifiers, file names, paths, and symbols keep original spelling.
    
    ## Before Generation: Ask Three Questions
    
    Unless already specified:
    
    1. **Mode**: `Learning` (read-only study) or `Maintenance` (active development).
    2. **Sub-agents**: `Yes, max 3` (recommended), custom limit, or `No`.
    3. **Ignore rules**: `Defaults + .gitignore` (recommended), or add custom patterns.
    
    Mode differences:
    
    | Aspect | Learning | Maintenance |
    |---|---|---|
    | Frontmatter | `mode: learning` | `mode: maintenance`, `commit: <hash>` |
    | Task Guide | suggested entry point | primary navigation, strict |
    | Domain | soft focus hint | hard filter unless justified |
    | Dependencies | reference material | gated by interface/impact rules |
    | Updates | one-time | incremental after code changes |
    
    ## Ignore Rules
    
    Merge in order:
    1. Built-ins: `.git/`, dependency dirs, virtualenvs, build outputs, caches, logs, lockfiles, minified files, binaries, image/font assets, IDE folders.
    2. Project `.gitignore`.
    3. User custom patterns.
    
    Include generated code only if it affects navigation; mark `Generated, do not edit manually`.
    
    Bijection applies only within the declared inventory scope. Low-priority areas may be deliberately excluded or represented only by a directory boundary, according to the user's priorities and project conventions. Record these choices visibly in the root CODEMAP's **Scope and Exclusions** section; never silently omit them.
    
    Use `Path / pattern | Treatment | Reason / basis` rows. Treatment is either `Excluded` (no inventory obligation inside this scope) or `Boundary only` (retain one directory entry marked `Internals not indexed`, without a child map or internal file inventory). Patterns are root-relative; list any included exceptions explicitly. Record default exclusions and `.gitignore` as identifiable rule sources, grouping routine patterns rather than listing every ignored file. Project-specific omissions need concrete paths/patterns and reasons, not merely “defaults”. This section is authoritative; frontmatter `ignore` summarizes or points to it.
    
    Ignoring indexing does not mean code is obsolete, safe to delete, or exempt from correctness checks. If a task touches an excluded area or depends on its contract, inspect what is needed and reconsider the recorded scope. Update the scope decision if it changes; do not claim unindexed internals were verified.
    
    ## Generation Workflow
    
    ### 1. Build Global Context
    
    Read lightweight project context only:
    - Prefer root `README.md` / `README.rst` / `README.txt`.
    - Else metadata: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, etc.
    - Else infer from structure; mark guesses `inferred, verify against code`.
    
    Produce: purpose, architecture shape, major domains. Omit badges/changelogs.
    
    ### 2. Scan and Measure
    
    After applying ignore rules:
    - Build filtered directory topology.
    - Count source files, lines, size per first-level subdirectory and project total.
    - Files over 1000 lines:
      - `<=5` → generate all `.analysis.md`.
      - `>5` → ask: all, top 5, selected, or none.
    
    ### 3. Dispatch Work
    
    If sub-agents enabled, choose count `K`:
    
    | Project size | K |
    |---|---|
    | `<=3000` lines or `<=500KB` | 1 |
    | `3001-15000` lines or `500KB-3MB` | `min(N, 2)` |
    | `>15000` lines or `>3MB` | `N` |
    
    If line count and size disagree, use the larger `K`. Assign first-level directories by greedy bin packing. Keep root loose files with main agent if small (`<=200` lines), else assign to lightest bin.
    
    Sub-agent prompts: self-contained, plain English. Include compressed global context, output language, ignore rules, assigned directories, large files list, required formats, forbidden paths. No overlapping write ownership.
    
    ### 4. Generate Per-Directory Maps
    
    One `CODEMAP.md` per fully indexed source directory (root + included subdirectories). Excluded and boundary-only interiors do not require maps. Each map describes only the current directory level.
    
    ### 5. Assemble Root and Install Protocol
    
    Read first-level CODEMAP summaries and Task Guides. Write root `CODEMAP.md`, then install the Navigation Protocol block into `AGENTS.md` (preferred) or `CLAUDE.md`. Replace existing block if present; do not append duplicates. If neither file exists, create `AGENTS.md`.
    
    ---
    
    ## CODEMAP.md Structure
    
    ### Frontmatter
    
    ```yaml
    ---
    mode: learning | maintenance
    commit: abc1234f        # maintenance only
    ignore: ...             # root only
    generated_at: YYYY-MM-DD
    stats:                  # root only
      total_files: 114
      total_lines: 18200
      total_size: 4.2 MB
    ---
    ```
    
    ### Sections
    
    Sections appear in this fixed order. Omit a section when it would be empty.
    
    | Section | Root | Mid-level | Leaf | Container |
    |---|---|---|---|---|
    | Summary | 1 sentence | 1 sentence | 1 sentence | 1 sentence |
    | Scope and Exclusions | yes | — | — | root only |
    | Task Guide | yes | yes | yes | yes |
    | Subdirectories | yes | yes | — | yes |
    | Key Exports | yes | yes | if needed | — |
    | Files | yes | yes | yes | — |
    | File Dependencies | — | immediate files only | yes | — |
    
    **Container directory** = directory with no source files, only subdirectories. Generate only Summary + Task Guide + Subdirectories.
    Root containers also retain Scope and Exclusions, so deliberate omissions remain visible.
    
    ### Summary
    
    One sentence. Mark uncertain inference with `(inferred)`.
    
    ### Task Guide
    
    Columns: `Task | Domain | Target | Also Check`.
    
    Rules:
    - Each row is a concrete scenario, not a vague category.
    - Learning mode: understanding intents (`理解认证流程`). Maintenance mode: modification intents (`新增认证方式`).
    - `Domain` must match values used in the same map's Files or Subdirectories tables.
    - `Target` = primary read set. `Also Check` = conditional candidate, not automatic read list. Keep short and empirical.
    - Ensure every functional subdirectory and every common modification scenario has at least one row.
    
    Task Guide interpretation by mode:
    
    - **Maintenance**: Read `Target` first. Read `Also Check` only when the task explicitly mentions it, target code proves it needed, or public contract impact requires it. If no row matches, filter by `Domain`; non-matching domains excluded unless justified.
    - **Learning**: Use `Target` as starting point. Treat `Also Check` as optional reference.
    
    ### Subdirectories
    
    Columns: `Dir | Domain | Depends On | Purpose`.
    
    Rules:
    - `Domain`: short functional area (e.g., `Auth`, `Prompt`, `Tool System`, `MCP`, `Runtime`).
    - **Domain granularity**: each functionally distinct subdirectory MUST have a unique Domain value. Never assign a single generic Domain (e.g., `LLM Integration`) to all entries.
    - `Depends On`: directory-level dependencies (internal and external). Use `—` if none.
    - `Purpose`: one sentence.
    
    ### Key Exports
    
    Columns: `Symbol | Source | Line`.
    
    Rules:
    - Only symbols used by **other directories**. Internal-only symbols belong in child CODEMAPs.
    - Line numbers as `L:<number>`. Sort by architectural importance.
    - Cap at ~15 entries. Child CODEMAPs handle detailed symbols.
    
    ### Files
    
    Columns vary by directory type:
    
    - **Root**: `File | Domain | Function`
    - **Mid-level / Leaf**: `File | Domain | Deps | Function`
    
    `Deps` column (compact notation):
    - `←` = files outside this directory that this file imports.
    - `→` = files outside this directory that depend on this file.
    - Example: `← core/errors.py | → main.py, chat/service.py`
    - `→` >5 files: `→ N files (foundational); rg "SymbolName" --type py -l`
    - Omit `Deps` column entirely when no file in the directory has cross-dir dependencies. Use `—` for individual files with no deps.
    
    Rules:
    - One row per immediate file. **Never list files from child directories that have their own CODEMAP.**
    - `Domain` must match local Task Guide / Subdirectories values. `—` for trivial re-export files.
    - `Function`: one concise sentence. For large files, append `→ see <filename>.analysis.md`.
    - Include pure re-export files such as `__init__.py` once in Files; Key Exports does not replace their canonical inventory row.
    
    ### File Dependencies
    
    Columns: `File | Imports (in-dir) | Exposed To (in-dir)`.
    
    Rules:
    - Same-directory relationships only. Only for immediate files (not child directory files).
    - Mid-level directories: list only files directly in the directory itself, never files in child subdirectories.
    - Maintenance mode: read `Imports` only when an imported interface contract is unclear. Read `Exposed To` only when changing a public signature, return type, or documented semantics.
    - Learning mode: reference only; do not chain-read unless current logic is unclear without it.
    
    ### Parent-Child Decoupling
    
    When a child directory has its own CODEMAP:
    - Parent does NOT list the child's internal files in Files, File Dependencies, or Key Exports.
    - Parent only references the child directory in Subdirectories and Task Guide.
    - Child-internal symbols appear only in the child's Key Exports.
    
    ---
    
    ## Large File Analysis (.analysis.md)
    
    For source files over 1000 lines, create `<filename>.analysis.md` beside it.
    
    ### Structure
    
    ```markdown
    ---
    source: filename.py
    lines: 1842
    generated_at: YYYY-MM-DD
    ✓ • 0ms

## Assistant

现有脚本已经能比较审查前后的文件哈希，但它主要返回变更清单，还缺少“接下来读哪些文件、读哪几段”的能力。适合补一层有输出预算的上下文工具；结构化搜索和语言服务器则按任务需要调用，避免维护一套始终加载的大索引。我会查官方文档，并用本地项目做一个可运行的方案。

## Activity

    $ python3 scripts/maintenance_state.py inspect --root .; rg --version; git --version; python3 --version; sed -n '1,120p' /home/fuurin/.codex/skills/.system/skill-creator/scripts/quick_validate.py
    Project: /home/fuurin/code/skills
    Status: review-required
    files: 70 monitored
      added: 70
        '.gitignore'
        'README.md'
        'codemap/LICENSE'
        'codemap/README.md'
        'codemap/README_EN.md'
        'codemap/SKILL.md'
        'performance-gradient-optimization/SKILL.md'
        'performance-gradient-optimization/agents/openai.yaml'
        'project-maintenance/SKILL.md'
        'project-maintenance/agents/openai.yaml'
        'project-maintenance/references/verification.md'
        'project-maintenance/scripts/maintenance_state.py'
        'the missing semester/agent-context-management/SKILL.md'
        'the missing semester/agent-context-management/exercises-dsh.md'
        'the missing semester/agent-context-management/exercises.md'
        'the missing semester/agentic-coding/SKILL.md'
        'the missing semester/agentic-coding/exercises-dsh.md'
        'the missing semester/agentic-coding/exercises.md'
        'the missing semester/ai-etiquette/SKILL.md'
        'the missing semester/asking-good-questions/SKILL.md'
        'the missing semester/asking-good-questions/exercises.md'
        'the missing semester/ci-cd/SKILL.md'
        'the missing semester/ci-cd/exercises.md'
        'the missing semester/code-quality-tooling/SKILL.md'
        'the missing semester/code-quality-tooling/exercises.md'
        'the missing semester/code-review/SKILL.md'
        'the missing semester/code-review/exercises.md'
        'the missing semester/containers-deployment/SKILL.md'
        'the missing semester/containers-deployment/exercises.md'
        'the missing semester/contributing-upstream/SKILL.md'
        'the missing semester/contributing-upstream/exercises.md'
        'the missing semester/debugging/SKILL.md'
        'the missing semester/debugging/exercises.md'
        'the missing semester/dev-environment/SKILL.md'
        'the missing semester/dev-environment/exercises.md'
        'the missing semester/dotfiles/SKILL.md'
        'the missing semester/dotfiles/exercises.md'
        'the missing semester/git-cli/SKILL.md'
        'the missing semester/git-cli/exercises.md'
        'the missing semester/git-data-model/SKILL.md'
        ... use --json for the complete list
    maps: 0 monitored
    ripgrep 15.2.0 (rev e89fff89ac)
    
    features:+pcre2
    simd(compile):+SSE2,-SSSE3,-AVX2
    simd(runtime):+SSE2,+SSSE3,+AVX2
    
    PCRE2 10.45 is available (JIT is available)
    git version 2.55.0
    Python 3.14.7
    #!/usr/bin/env python3
    """
    Quick validation script for skills - minimal version
    """
    
    import re
    import sys
    from pathlib import Path
    
    import yaml
    
    MAX_SKILL_NAME_LENGTH = 64
    
    
    def validate_skill(skill_path):
        """Basic validation of a skill"""
        skill_path = Path(skill_path)
    
        skill_md = skill_path / "SKILL.md"
        if not skill_md.exists():
            return False, "SKILL.md not found"
    
        content = skill_md.read_text()
        if not content.startswith("---"):
            return False, "No YAML frontmatter found"
    
        match = re.match(r"^---\n(.*?)\n---", content, re.DOTALL)
        if not match:
            return False, "Invalid frontmatter format"
    
        frontmatter_text = match.group(1)
    
        try:
            frontmatter = yaml.safe_load(frontmatter_text)
            if not isinstance(frontmatter, dict):
                return False, "Frontmatter must be a YAML dictionary"
        except yaml.YAMLError as e:
            return False, f"Invalid YAML in frontmatter: {e}"
    
        allowed_properties = {"name", "description", "license", "allowed-tools", "metadata"}
    
        unexpected_keys = set(frontmatter.keys()) - allowed_properties
        if unexpected_keys:
            allowed = ", ".join(sorted(allowed_properties))
            unexpected = ", ".join(sorted(unexpected_keys))
            return (
                False,
                f"Unexpected key(s) in SKILL.md frontmatter: {unexpected}. Allowed properties are: {allowed}",
            )
    
        if "name" not in frontmatter:
            return False, "Missing 'name' in frontmatter"
        if "description" not in frontmatter:
            return False, "Missing 'description' in frontmatter"
    
        name = frontmatter.get("name", "")
        if not isinstance(name, str):
            return False, f"Name must be a string, got {type(name).__name__}"
        name = name.strip()
        if name:
            if not re.match(r"^[a-z0-9-]+$", name):
                return (
                    False,
                    f"Name '{name}' should be hyphen-case (lowercase letters, digits, and hyphens only)",
                )
            if name.startswith("-") or name.endswith("-") or "--" in name:
                return (
                    False,
                    f"Name '{name}' cannot start/end with hyphen or contain consecutive hyphens",
                )
            if len(name) > MAX_SKILL_NAME_LENGTH:
                return (
                    False,
                    f"Name is too long ({len(name)} characters). "
                    f"Maximum is {MAX_SKILL_NAME_LENGTH} characters.",
                )
    
        description = frontmatter.get("description", "")
        if not isinstance(description, str):
            return False, f"Description must be a string, got {type(description).__name__}"
        description = description.strip()
        if description.startswith("[TODO:"):
            return False, "Description contains an unfinished TODO placeholder"
        if description:
            if "<" in description or ">" in description:
                return False, "Description cannot contain angle brackets (< or >)"
            if len(description) > 1024:
                return (
                    False,
                    f"Description is too long ({len(description)} characters). Maximum is 1024 characters.",
                )
    
        body = content[match.end() :]
        fence_marker = None
        fence_length = 0
        for line in body.splitlines():
            fence = re.match(r"^[ \t]*(?:(?:[-+*]|\d+[.)])[ \t]+)?(`{3,}|~{3,})(.*)$", line)
            if fence:
                marker = fence.group(1)
                if fence_marker is None:
                    fence_marker = marker[0]
                    fence_length = len(marker)
                elif (
                    marker[0] == fence_marker
                    and len(marker) >= fence_length
                    and not fence.group(2).strip()
                ):
                    fence_marker = None
                    fence_length = 0
                continue
    
            if fence_marker is None and re.fullmatch(
                r"[ ]{0,3}\[TODO:[^\n]*\][ \t]*", line
            ):
                return False, "Skill instructions contain an unfinished TODO placeholder"
    
        return True, "Skill is valid!"
    
    
    if __name__ == "__main__":
    ✓ • 0ms

## Activity

    Searched the web for site.ast-grep.github.io guide CLI json output rewrite, site.github.com BurntSushi ripgrep GUIDE json output files, site.universal-ctags.io output formats json incremental, site.clangd.llvm.org design code index background indexing

## Activity

    $ npx --yes ctx7@latest library ast-grep 'CLI structural search with JSON output and rewrite dry run'
    
    1. Title: Ast-grep
       Context7-compatible library ID: /ast-grep/ast-grep
       Description: ast-grep is a powerful CLI tool for structural code search, linting, and rewriting, allowing users to match and manipulate code based on abstract syntax tree patterns.
       Code Snippets: 781
       Source Reputation: High
       Benchmark Score: 68.97
    
    2. Title: Ast-Grep
       Context7-compatible library ID: /ast-grep/ast-grep.github.io
       Description: ast-grep (sg) is a fast and user-friendly tool for large-scale code searching, linting, and rewriting.
       Code Snippets: 2318
       Source Reputation: High
       Benchmark Score: 80.43
    
    3. Title: Ast-Grep
       Context7-compatible library ID: /llmstxt/ast-grep_github_io_llms_txt
       Description: Ast-Grep is a tool that allows for fast and powerful structural search, replace, and rewrite of code using Abstract Syntax Trees, helping developers refactor and maintain large codebases efficiently.
       Code Snippets: 1856
       Source Reputation: High
       Benchmark Score: 84.8
    
    4. Title: ast-grep
       Context7-compatible library ID: /llmstxt/ast-grep_github_io_llms-full_txt
       Description: ast-grep is a powerful linter and code formatter that leverages Abstract Syntax Trees (AST) for precise code analysis and manipulation across various programming languages.
       Code Snippets: 3776
       Source Reputation: High
       Benchmark Score: 78.26
    
    5. Title: ast-grep Agent Skill
       Context7-compatible library ID: /ast-grep/agent-skill
       Description: ast-grep Agent Skill provides plugins and skills for AI coding agents to perform structural code search and analysis using Abstract Syntax Tree patterns.
       Code Snippets: 111
       Source Reputation: High
    
    ✓ • 3.80s

## Activity

    Opened https://ast-grep.github.io/reference/cli.html

## Activity

    $ rg --help | rg -n -A 9 -- '--json|--no-config|--hidden|--files|--fixed-strings'; git ls-files -z project-maintenance | python3 -c 'import sys; print("\n".join(x.decode() for x in sys.stdin.buffer.read().split(b"\0") if x))'; wc -l SKILL.md scripts/maintenance_state.py references/verification.md
    16:    rg [OPTIONS] --files [PATH ...]
    17-    rg [OPTIONS] --type-list
    18-    command | rg [OPTIONS] PATTERN
    19-    rg [OPTIONS] --help
    20-    rg [OPTIONS] --version
    21-
    22-POSITIONAL ARGUMENTS:
    23-    <PATTERN>
    24-        A regular expression used for searching. To match a pattern beginning
    25-        with a dash, use the -e/--regexp flag.
    --
    244:    -F, --fixed-strings
    245-        Treat all patterns as literals instead of as regular expressions. When
    246-        this flag is used, special regular expression meta characters such as
    247-        .(){}*+ should not need be escaped.
    248-
    249-        This flag can be disabled with --no-fixed-strings.
    250-
    251-    -i, --ignore-case
    252-        When this flag is provided, all patterns will be searched case
    253-        insensitively. The case insensitivity rules used by ripgrep's default
    --
    268:        combining this flag with -l/--files-with-matches will emit files that
    269-        contain any lines that do not match the patterns given. That's not the
    270:        same as, for example, --files-without-match, which will emit files that
    271-        do not contain any matching lines.
    272-
    273-        This flag can be disabled with --no-invert-match.
    274-
    275-    -x, --line-regexp
    276-        When enabled, ripgrep will only show matches surrounded by line
    277-        boundaries. This is equivalent to surrounding every pattern with ^ and
    278-        $. In other words, this only prints lines where the entire line
    279-        participates in a match.
    --
    625:    -., --hidden
    626-        Search hidden files and directories. By default, hidden files and
    627-        directories are skipped. Note that if a hidden file or a directory is
    628-        whitelisted in an ignore file, then it will be searched even if this
    629-        flag isn't provided. Similarly if a hidden file or directory is given
    630-        explicitly as an argument to ripgrep.
    631-
    632-        A file or directory is considered hidden if its base name starts with a
    633-        dot character (.). On operating systems which support a "hidden" file
    634-        attribute, like Windows, files with this attribute are also considered
    --
    637:        Note that -./--hidden will include files and folders like .git
    638-        regardless of --no-ignore-vcs. To exclude such paths when using
    639:        -./--hidden, you must explicitly ignore them using another flag or
    640-        ignore file.
    641-
    642-        This flag can be disabled with --no-hidden.
    643-
    644-    --iglob=GLOB
    645-        Include or exclude files and directories for searching that match the
    646-        given glob. This always overrides any other ignore logic. Multiple glob
    647-        flags may be used. Globbing rules match .gitignore globs. Precede a
    648-        glob with a ! to exclude it. If multiple globs match a file or
    --
    711:        whose names begin with a dot. For that, see the -./--hidden flag. This
    712-        flag also does not impact whether filter rules from .gitignore files
    713-        are respected.
    714-
    715-        This flag can be disabled with --ignore-dot.
    716-
    717-    --no-ignore-exclude
    718-        Don't respect filter rules from files that are manually configured for
    719-        the repository. For example, this includes git's .git/info/exclude.
    720-
    --
    754:        are affected by -./--hidden and its related flags instead.
    755-
    756-        This flag implies --no-ignore-parent for source control ignore files as
    757-        well.
    758-
    759-        This flag can be disabled with --ignore-vcs.
    760-
    761-    --no-require-git
    762-        When this flag is given, source control ignore files such as .gitignore
    763-        are respected even if no git repository is present.
    --
    864:        -u/--unrestricted flags is equivalent to --no-ignore -./--hidden. Three
    865:        -u/--unrestricted flags is equivalent to --no-ignore -./--hidden
    866-        --binary.
    867-
    868-        The only filtering ripgrep still does when -uuu is given is to skip
    869-        symbolic links and to avoid printing matches from binary files.
    870-        Symbolic links can be followed via the -L/--follow flag, and binary
    871-        files can be treated as text files via the -a/--text flag.
    872-
    873-OUTPUT OPTIONS:
    874-    -A NUM, --after-context=NUM
    --
    925:        --vimgrep and --json.
    926-
    927-        The possible values for this flag are:
    928-
    929-        never: Colors will never be used.
    930-
    931-        auto: The default. ripgrep tries to be smart.
    932-
    933-        always: Colors will always be used regardless of where output is sent.
    934-
    --
    1224:        of matching files such as with -c/--count, -l/--files-with-matches and
    1225:        --files. This option is useful for use with xargs.
    1226-
    1227-    -o, --only-matching
    1228-        Print only the matched (non-empty) parts of a matching line, with each
    1229-        such part on a separate output line.
    1230-
    1231-    --path-separator=SEPARATOR
    1232-        Set the path separator to use when printing file paths. This defaults
    1233-        to your platform's path separator, which is / on Unix and \ on Windows.
    1234-        This flag is intended for overriding the default when the environment
    --
    1268:        When --files is used, ripgrep will stop finding files after finding the
    1269-        first file that does not match any ignore rules.
    1270-
    1271-    -r REPLACEMENT, --replace=REPLACEMENT
    1272-        Replaces every match with the text given when printing results. Neither
    1273-        this flag nor any other ripgrep flag will modify your files.
    1274-
    1275-        Capture group indices (e.g., $5) and names (e.g., $foo) are supported
    1276-        in the replacement string. Capture group indices are numbered based on
    1277-        the position of the opening parenthesis of the group, where the
    --
    1383:        from ripgrep, such as via the --json flag. One alternative to avoiding
    1384-        exorbitant memory usage is to force ripgrep into single threaded mode
    1385-        with the -j/--threads flag. Note though that this will not impact the
    1386-        total size of the output, just the heap memory that ripgrep will use.
    1387-
    1388-    -H, --with-filename
    1389-        This flag instructs ripgrep to print the file path for each matching
    1390-        line. This is the default when more than one file is searched. If
    1391-        --heading is enabled (the default when printing to a tty), the file
    1392-        path will be shown above clusters of matches from each file; otherwise,
    --
    1432:        with the output of -l/--files-with-matches. Notably, by default,
    1433-        ripgrep tries to avoid searching files with binary data. With this
    1434-        flag, ripgrep needs to search the entire content of files, which may
    1435:        include binary data. But with -l/--files-with-matches, ripgrep can stop
    1436-        as soon as a match is observed, which may come well before any binary
    1437-        data. To avoid this inconsistency without disabling binary detection,
    1438-        use the --binary flag.
    1439-
    1440-        This overrides the --count-matches flag. Note that when -c/--count is
    1441-        combined with -o/--only-matching, then ripgrep behaves as if
    1442-        --count-matches was given.
    1443-
    1444-    --count-matches
    --
    1459:    -l, --files-with-matches
    1460-        Print only the paths with at least one match and suppress match
    1461-        contents.
    1462-
    1463-        Note that it is possible for this flag to have results inconsistent
    1464-        with the output of -c/--count. Notably, by default, ripgrep tries to
    1465-        avoid searching files with binary data. With this flag, ripgrep might
    1466-        stop searching before the binary data is observed. But with -c/--count,
    1467-        ripgrep has to search the entire contents to determine the match count,
    1468-        which means it might see binary data that causes it to skip searching
    --
    1472:        This overrides --files-without-match.
    1473-
    1474:    --files-without-match
    1475-        Print the paths that contain zero matches and suppress match contents.
    1476-
    1477:        This overrides -l/--files-with-matches.
    1478-
    1479:    --json
    1480-        Enable printing results in a JSON Lines format.
    1481-
    1482-        When this flag is provided, ripgrep will emit a sequence of messages,
    1483-        each encoded as a JSON object, where there are five different message
    1484-        types:
    1485-
    1486-        begin: A message that indicates a file is being searched and contains
    1487-        at least one match.
    1488-
    --
    1511:        as --files, -l/--files-with-matches, --files-without-match, -c/--count
    1512-        or --count-matches. ripgrep will report an error if any of the
    1513:        aforementioned flags are used in concert with --json.
    1514-
    1515-        Other flags that control aspects of the standard output such as
    1516-        -o/--only-matching, --heading, -r/--replace, -M/--max-columns, etc.,
    1517:        have no effect when --json is set. However, enabling JSON output will
    1518-        always implicitly and unconditionally enable --stats.
    1519-
    1520-        A more complete description of the JSON format used can be found here:
    1521-        https://docs.rs/grep-printer/*/grep_printer/struct.JSON.html.
    1522-
    1523-        This flag can be disabled with --no-json.
    1524-
    1525-LOGGING OPTIONS:
    1526-    --debug
    --
    1560:        This flag is always and implicitly enabled when --json is used.
    1561-
    1562:        Note that this flag has no effect if --files, -l/--files-with-matches
    1563:        or --files-without-match is passed.
    1564-
    1565-        This flag can be disabled with --no-stats.
    1566-
    1567-    --trace
    1568-        Show trace messages. This shows even more detail than the --debug flag.
    1569-        Generally, one should only use this if --debug doesn't emit the
    1570-        information you're looking for.
    1571-
    1572-OTHER BEHAVIORS:
    1573:    --files
    1574-        Print each file that would be searched without actually performing the
    1575-        search. This is useful to determine whether a particular file is being
    1576-        searched or not.
    1577-
    1578-        This overrides --type-list.
    1579-
    1580-    --generate=KIND
    1581-        This flag instructs ripgrep to generate some special kind of output
    1582-        identified by KIND and then quit without searching. KIND can be one of
    --
    1597:    --no-config
    1598-        When set, ripgrep will never read configuration files. When this flag
    1599-        is present, ripgrep will not respect the RIPGREP_CONFIG_PATH
    1600-        environment variable.
    1601-
    1602-        If ripgrep ever grows a feature to automatically read configuration
    1603-        files in pre-defined locations, then this flag will also disable that
    1604-        behavior as well.
    1605-
    1606-    --pcre2-version
    
       90 SKILL.md
      245 scripts/maintenance_state.py
       47 references/verification.md
      382 total
    ✓ • 0ms

## Activity

    $ npx --yes ctx7@latest docs /ast-grep/ast-grep.github.io 'Command line search --json=stream and rewrite preview without --update-all; outline symbols'
    ### JSON file object for outline output
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/reference/cli/outline.md
    
    Example of a single file object emitted by `--json` and `--json=compact` (as an array of file objects) and by `--json=stream` (one file object per line).
    
    ```json
    {
      "path": "src/parser.ts",
      "language": "TypeScript",
      "items": [
        {
          "name": "Parser",
          "symbolType": "class",
          "role": "item",
          "isImport": false,
          "isExported": true,
          "range": {
            "byteOffset": { "start": 1200, "end": 2500 },
            "start": { "line": 39, "column": 0 },
            "end": { "line": 97, "column": 1 }
          },
          "signature": "export class Parser",
          "astKind": "class_declaration",
          "members": [
            {
              "name": "parse",
              "symbolType": "method",
              "role": "member",
              "isPublic": true,
              "range": {
                "byteOffset": { "start": 1300, "end": 1900 },
                "start": { "line": 43, "column": 2 },
                "end": { "line": 71, "column": 3 }
              },
              "signature": "parse(...)",
              "astKind": "method_definition"
            }
          ]
        }
      ]
    }
    ```
    
    --------------------------------
    
    ### Example output of --json=stream
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/guide/tools/json.md
    
    Shows the output format of --json=stream: each match is printed as a separate JSON object on its own line. This allows streaming to programs that read one object per line. Note that --json=stream requires the equal sign; --json stream is parsed as --json=pretty stream.
    
    ```bash
    $ ast-grep -p pattern --json=stream
    {"text":"Some(matched)", ... }
    {"text":"Some(matched)", ... }
    {"text":"Some(matched)", ... }
    ```
    
    --------------------------------
    
    ### Use JSON output with --json, --json=compact, and --json=stream
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/guide/outline-code.md
    
    Use JSON output for piping, filtering, or programmatic comparison. --json=stream prints one JSON object per file, convenient for large directories. The jq example filters for function symbols.
    
    ```shell
    ast-grep outline src/parser.ts --json
    ast-grep outline src --json=compact
    ast-grep outline src --json=stream
    ```
    
    ```shell
    ast-grep outline src --json=stream |
      jq '.items[] | select(.symbolType == "function") | .name'
    ```
    
    --------------------------------
    
    ### Filter outline items with --match, --type, and --pub-members
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/guide/outline-code.md
    
    Filter top-level outline items by regular expression with --match, by symbol type with --type, or both. Use --pub-members to show only public members. These flags apply only to top-level items; member display is controlled by --view.
    
    ```shell
    ast-grep outline src/parser.ts --match Parser
    ```
    
    ```shell
    ast-grep outline crates --type struct,enum,interface
    ```
    
    ```shell
    ast-grep outline src/parser.ts --match Parser --type class --view expanded
    ```
    
    ```shell
    ast-grep outline src/parser.ts --match Parser --view expanded --pub-members
    ```
    
    ### Rewrite Code > Using `ast-grep run -p 'pat' --rewrite`
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/guide/rewrite-code.md
    
    The simplest way to rewrite code is to use the `--rewrite` flag with the `ast-grep run` command. This flag takes a string argument that specifies the new code to replace the matched pattern.
    For example, if you want to change all occurrences of identifier `foo` to `bar`, you can run:
    
    ```bash
    ast-grep run --pattern 'foo' --rewrite 'bar' --lang python
    ```
    
    This will show you a diff of the changes that will be made. If you are using interactive mode by the `--interactive` flag, ast-grep ask you if you want to apply them.
    
    :::tip
    You can also use the `--update-all` or `-U` flag to automatically accept the changes without confirmation.
    :::
    ✓ • 4.44s

## Activity

    Opened https://ast-grep.github.io/guide/tooling-overview.html

## User

这个和code map对照一下，防止重复造轮子

## Assistant

我会先和 codemap 对照职责及现有资源。目录导航、索引范围和地图格式沿用 codemap；project-maintenance 负责变更检测、影响判断和退役流程。对照后再确定需要补哪些工具。

## Activity

    $ readlink -f /home/fuurin/.codex/skills/codemap; readlink -f /home/fuurin/.codex/skills/project-maintenance; wc -l ../codemap/SKILL.md; sed -n '225,400p' ../codemap/SKILL.md; sed -n '1,170p' ../codemap/README.md; git diff -- ../codemap/SKILL.md; cat ../.gitignore
    /home/fuurin/code/skills/codemap
    /home/fuurin/code/skills/project-maintenance
    337 ../codemap/SKILL.md
    
    ```markdown
    ---
    source: filename.py
    lines: 1842
    generated_at: YYYY-MM-DD
    ---
    
    > One-sentence summary.
    
    ## Feature Index
    
    | Intent | Lines | Notes |
    |---|---|---|
    
    ## Symbols
    
    | Symbol | Type | Line |
    |---|---|---|
    
    ## Logical Sections
    
    | Lines | Content |
    |---|---|
    ```
    
    ### Rules
    
    - **Feature Index** is primary. Map concrete learning or development intents to exact line ranges. Notes: same-file coupling only. If no intent maps to a section, omit that row.
    - **Symbols**: top-level public symbols only (classes, functions, constants). Not internal helpers.
    - **Logical Sections**: high-level structural segments only, **5-10 rows max**. Provides structural overview and serves as fallback when Feature Index has no match. Do not expand to function-level granularity.
    - Optional **Class Hierarchy** only when inheritance depth >2.
    - No code snippets, API signatures, or implementation detail paragraphs.
    - In maintenance mode, agents read only matched line ranges from Feature Index; Logical Sections is the fallback.
    
    ---
    
    ## Navigation Protocol (Project-Level Injection)
    
    Install exactly one protocol block into project `AGENTS.md` or `CLAUDE.md`. Choose by mode.
    
    ### Learning Mode Block
    
    ````markdown
    ## CODEMAP Navigation Protocol
    
    This project uses hierarchical `CODEMAP.md` index files for code navigation. Files over 1000 lines may have companion `.analysis.md` structural maps.
    
    ### Navigation Rules
    
    1. Start from root `CODEMAP.md`. Read Task Guide first.
    2. Task Guide match: Target = primary read set. Also Check = conditional candidates (decide after reading Target).
    3. No Task Guide match → filter Subdirectories by Domain, enter only matching-domain subdirectories.
    4. Drill down layer by layer; consult local Task Guide at each level before reading source files.
    5. Container directories (no source files): read only Task Guide + Subdirectories.
    6. Large files: read `.analysis.md` Feature Index first, match Intent to line ranges. Use Logical Sections as fallback.
    7. Batch-read final target files in parallel.
    8. No speculative expansion: extend read set only when already-read code proves the need.
    ````
    
    ### Maintenance Mode Block
    
    ````markdown
    ## CODEMAP Navigation Protocol
    
    This project uses hierarchical `CODEMAP.md` index files for code navigation. Files over 1000 lines may have companion `.analysis.md` structural maps. For development tasks, these rules are strict navigation constraints.
    
    ### Navigation Rules
    
    1. Start from root `CODEMAP.md`. Read Task Guide first.
    2. Task Guide match: Target = primary read set. Read Also Check only when the task explicitly involves it, target code proves the need, or public contract impact requires it.
    3. No Task Guide match → filter Subdirectories by Domain. Non-matching domains are excluded unless already-read code gives a concrete reason.
    4. Drill down layer by layer; consult local Task Guide at each level before reading source files.
    5. Container directories (no source files): read only Task Guide + Subdirectories.
    6. Large files: read `.analysis.md` Feature Index first, match Intent to line ranges. Use Logical Sections as fallback.
    7. Batch-read final target files in parallel.
    8. No speculative expansion: each additional file requires an explicit reason.
    
    ### Dependency Gating
    
    The Deps column in Files tables marks cross-directory dependencies:
    - `←` (imports): read only when the imported interface contract is needed to understand the current file.
    - `→` (exposed to) ≤5 files: read only when changing a public signature, return type, or documented semantics.
    - `→` >5 files (foundational): run the search command provided in CODEMAP, filter by Domain, then read only justified matches.
    - No chaining: do not read dependencies-of-dependencies unless a specific contract gap remains.
    
    ### Update Rules
    
    Every code change requires evaluating all three operations within the affected task scope; perform each applicable operation without waiting for a separate cleanup request:
    - Add: supply missing implementation or contract coverage and missing map entries for in-scope files/directories.
    - Delete: remove confirmed redundant or superseded implementations and their exclusive tests, dependencies, build routes, documentation, and stale map/analysis entries. Check real consumers, supported platforms, dynamic/FFI entry points, and public contracts first.
    - Modify: update existing implementations and canonical entries in place when responsibilities, interfaces, dependencies, paths, or analysis line ranges change. Moves/renames remove old routes and install current ones, including parent Task Guide/Subdirectories references.
    
    Completion requires a bijection between included source files/directories and their canonical local Files/Subdirectories entries: each entity appears exactly once, each entry resolves to a current entity, and excluded scope is explicit. Task Guide, Key Exports, and dependency links may reference the same entity; they must remain accurate without duplicating inventory ownership. Check source → map for missing/duplicate entries and map → source for stale entries and semantics, using the working tree rather than just the last commit.
    
    Deliberately unindexed areas must appear in root CODEMAP's Scope and Exclusions with root-relative paths/patterns, treatment (`Excluded` or `Boundary only`), and reason/basis. Boundary-only directories retain a marked directory entry, without an internal inventory obligation. Record rule sources for routine exclusions and explicit exceptions. When a task touches these areas or depends on their contracts, inspect the necessary code and reconsider scope; exclusion is neither deletion evidence nor a correctness exemption.
    
    Actively recheck retirement conditions for affected compatibility layers and migration code. Once replacement and consumer migration are verified, remove the old path in this task; retained exceptions need concrete consumers or a current contract and a removal condition. Do not accumulate `old/`, `v1/`, or `backup/` copies. Age or no local text references alone does not justify deletion.
    
    Pure internal equivalent changes may leave maps unchanged only after verifying that indexed responsibilities, contracts, dependencies, symbols, and line ranges remain accurate. Briefly report applicable additions/deletions/modifications, reasons for no action, and unresolved gaps; timestamps and passing builds alone do not establish map accuracy.
    ````
    
    ---
    
    ## Edge Cases
    
    - **Monorepo**: map each package root plus a top-level package index.
    - **Deep nesting**: layer-by-layer drill-down; each level's map stays local.
    - **Huge flat directory** (`>200` files): group Files rows by Domain subheadings.
    - **Generated code**: include only when navigation needs it; mark `Generated, do not edit manually`.
    - **No metadata/README**: infer cautiously; mark uncertainty.
    - **Task Guide gaps**: acceptable. Fall back to Domain filtering and Key Exports.
    - **Foundational files** (>5 dependents): provide grep command, require Domain filtering.
    # codemap-skill
    
    **中文** | **[English](README_EN.md)**
    
    ---
    
    为 Claude Code 及其他 AI 编程 Agent 设计的**层级化代码库导航索引** Skill。
    
    在项目根目录和每个源码子目录下生成 `CODEMAP.md` 索引文件，以及针对超大文件（>1000 行）的 `<filename>.analysis.md` 深度分析伴生文件。采用 Task Guide → Domain 过滤 → 依赖安全网的**约束式导航策略**，从源头防止 Agent 扩散性读取无关文件。
    
    ## 核心特性
    
    ### 索引与导航
    - **层级化索引**：根目录 + 每个子目录各自生成 `CODEMAP.md`，包含精简目录结构、文件/子目录功能概要
    - **Domain 功能域标注**：每个文件/子目录标注所属功能域（Auth/User Data/API 等），Agent 按域过滤排除无关文件
    - **Task Guide 任务路由**：预定义任务类型→目标文件的精确映射（如"新增 Loss 函数 → `losses.py`"），消除 Agent 自行语义扩展的不确定性
    - **Also Check 跨目录关联**：Task Guide 中附带经验性跨目录关联文件列表（非全量依赖图），确保关联文件不被遗漏
    
    ### 依赖关系管理（文件级 + 目录级）
    - **File Dependencies（同目录内）**：IMPORT → EXPOSED_TO 双向链表，仅在接口契约需要理解或公开签名变更时触发读取，禁止无条件链式遍历
    - **Cross-Dir Dependencies（跨目录）**：每个文件标注其跨目录 Imports 和 Exposed To。Exposed To ≤5 精确列出文件路径；>5 则降级为 grep 动态查询指令（"foundational"），防止基础文件的静态列表过时
    - **目录级 Dependencies**：标注 internal/external 以区分视野范围内外的依赖
    
    ### 大文件精确定位
    - **Feature Index（功能→行范围索引）**：`.analysis.md` 中新增意图→行范围映射表，Agent 直接匹配任务关键词定位需要读取的代码段
    - **Logical Sections（逻辑分段）**：作为 Feature Index 未匹配时的回退方案
    - **顶层符号表 + 类继承关系**：快速了解文件结构
    
    ### Agent 行为约束
    - **Two-Stage Read Protocol**：Stage 1 仅读取 Task Guide + Domain 匹配的文件；Stage 2 仅在分析证明确实需要时补充读取
    - **依赖读取条件门控**：Imports 仅在接口契约理解需要时读；Exposed To 仅在公开签名/语义变更时读；>5 foundational 文件先 grep 再按 Domain 过滤
    - **自动写入导航协议**：在 `CLAUDE.md` / `AGENTS.md` 中写入 10 条增强约束规则 + 决策树更新规则
    
    ### 工程特性
    - **并行 Sub-agent 生成**：按代码行数贪心装箱均衡负载
    - **双模式**：学习模式（一次生成）/ 维护模式（基于 `git diff` 增量更新 + 决策树自主判断更新范围）
    - **三层忽略规则**：内置默认 + `.gitignore` + 用户自定义
    - **多语言输出**：CODEMAP 内容语言跟随用户提问语言
    
    ## 生成的文件
    
    运行此 Skill 后，项目中会新增以下文件：
    
    ```
    project-root/
    ├── CODEMAP.md                          # 根目录索引（含 Task Guide + Domain + Dependencies）
    ├── CLAUDE.md (追加导航协议)             # 或 AGENTS.md
    ├── src/
    │   ├── CODEMAP.md                      # src/ 索引（含 Task Guide + Domain + Dependencies）
    │   ├── models/
    │   │   ├── CODEMAP.md                  # 含 Files(Domain+Cross-Dir Deps) + File Deps + Task Guide
    │   │   └── large_model.py.analysis.md  # 大文件分析（含 Feature Index + Logical Sections）
    │   └── utils/
    │       └── CODEMAP.md
    └── tests/
        └── CODEMAP.md
    ```
    
    ## 安装
    
    将 `SKILL.md` 复制到 Claude Code 的 skills 目录：
    
    ```bash
    mkdir -p ~/.claude/skills/codemap
    cp SKILL.md ~/.claude/skills/codemap/SKILL.md
    ```
    
    ## 使用
    
    在 Claude Code 会话中触发（以下任一方式）：
    
    - `/codemap`
    - 对 Claude 说 "帮我索引这个项目" / "map this codebase" / "generate codemap"
    
    Skill 会依次询问：
    1. 项目模式（学习 / 维护）
    2. 是否启用并行 sub-agent（默认上限 3）
    3. 额外忽略规则
    
    然后自动扫描、生成所有 CODEMAP.md 和分析文件。
    
    ## 导航工作流
    
    Agent 使用 CODEMAP 的**约束式读取流程**：
    
    ```
    Step 1: 读取根 CODEMAP → Task Guide 匹配当前任务类型
            ↓ 命中 → Target + Also Check 直接给出初始文件集
            ↓ 未命中 → Domain 过滤定位目标目录
    
    Step 2: 并行读取目标子目录 CODEMAP
            ↓ Task Guide 精确定位文件
            ↓ Domain 列排除非匹配文件
            ↓ File Dependencies 仅条件性触发（接口变更/契约理解）
    
    Step 3: 处理跨目录依赖
            ↓ Also Check → 直接读取（经验验证）
            ↓ Exposed To ≤5 → 仅在签名/语义变更时读
            ↓ Exposed To >5 (foundational) → grep → Domain 过滤 → 读
    
    Step 4: 编译最终文件集 → 去重
    
    Step 5: 大文件先读 .analysis.md Feature Index → 定位行范围
    
    Step 6: 并行批量读取所有目标源文件（使用 offset/limit 精准读取大文件行范围）
    
    Step 7 (Stage 2): 仅在分析证明确实需要时补充读取更多文件
    ```
    
    **核心效率原则**：阅读 3-4 个 CODEMAP（~200 行）即可精确定位 5-8 个源文件。Task Guide + Domain 过滤在读取任何源码之前缩小候选集。Feature Index 对大文件实现行级精准读取。依赖关系信息是**安全网而非读取指令**——仅在特定条件触发下才引发额外读取。
    
    ## 更新规则（维护模式）
    
    Agent 在修改代码后，通过以下**决策树**自主判断是否需要更新 CODEMAP：
    
    1. **结构变更**（文件/目录增删移）→ 重新生成受影响目录的全部 CODEMAP 内容
    2. **接口变更**（公开符号签名/名称变更）→ 更新 Key Exports、Cross-Dir Dependencies（含阈值穿越处理），向上传播
    3. **实现变更**（bug fix / 内部重构 / 参数调整）→ 无需更新 CODEMAP（仅在大文件行偏移 >20 行时更新 Logical Sections 行号）
    
    ## 许可证
    
    MIT
    diff --git a/codemap/SKILL.md b/codemap/SKILL.md
    index 73d729f..4de6f2f 100644
    --- a/codemap/SKILL.md
    +++ b/codemap/SKILL.md
    @@ -13,6 +13,16 @@ Generate hierarchical `CODEMAP.md` files that help agents locate relevant code w
     - Prefer positive guidance (Task Guide, Domain, Key Exports) over broad listings.
     - Dependencies are a safety net, not an invitation to chain-read.
     - Each CODEMAP describes only its own directory level. Child directory details belong in child CODEMAPs.
    +- Maintain a bijection between in-scope source entities and canonical inventory entries: no omissions, no duplicate ownership, no entries for nonexistent entities. Navigation references may be many-to-one; they are not additional inventory entries.
    +- Every code change requires considering addition, deletion, and modification together. Maintenance also actively retires superseded code within the authorized task scope; adding an index entry does not complete a migration.
    +
    +## Bijection and Freshness
    +
    +Define the inventory scope using the declared ignore rules. Every included immediate source file has exactly one canonical row in its directory's Files table; every included immediate child directory has exactly one Subdirectories row. Parent maps route to children without duplicating their inventories. CODEMAPs and companion analysis files are index artifacts, not source entities to recursively index. Key Exports and Task Guide are selective navigation views, not exhaustive inventories of all symbols or tasks.
    +
    +Validate both directions against the current source tree, including uncommitted changes: source → map catches omissions and duplicate ownership; map → source catches stale paths, symbols, responsibilities, dependencies, and line ranges. Do not exclude a source merely to hide an omission. Shared implementations can have multiple callers and navigation references without duplicate canonical entries.
    +
    +Freshness is a completion condition after each code change, not a timestamp update. In maintenance mode, inspect affected implementations for replacement, duplication, obsolete requirements, and completed migration scaffolding. Retire confirmed obsolete code and its exclusive tests, dependencies, build routes, documentation, and map entries in the same task. Retained compatibility paths need concrete consumers or a current contract and an explicit removal condition; recheck that condition when affected. Age or absence of local text references alone is not proof of obsolescence. Learning mode remains read-only for source code.
    
     ## Language Rule
    
    @@ -45,6 +55,12 @@ Merge in order:
    
     Include generated code only if it affects navigation; mark `Generated, do not edit manually`.
    
    +Bijection applies only within the declared inventory scope. Low-priority areas may be deliberately excluded or represented only by a directory boundary, according to the user's priorities and project conventions. Record these choices visibly in the root CODEMAP's **Scope and Exclusions** section; never silently omit them.
    +
    +Use `Path / pattern | Treatment | Reason / basis` rows. Treatment is either `Excluded` (no inventory obligation inside this scope) or `Boundary only` (retain one directory entry marked `Internals not indexed`, without a child map or internal file inventory). Patterns are root-relative; list any included exceptions explicitly. Record default exclusions and `.gitignore` as identifiable rule sources, grouping routine patterns rather than listing every ignored file. Project-specific omissions need concrete paths/patterns and reasons, not merely “defaults”. This section is authoritative; frontmatter `ignore` summarizes or points to it.
    +
    +Ignoring indexing does not mean code is obsolete, safe to delete, or exempt from correctness checks. If a task touches an excluded area or depends on its contract, inspect what is needed and reconsider the recorded scope. Update the scope decision if it changes; do not claim unindexed internals were verified.
    +
     ## Generation Workflow
    
     ### 1. Build Global Context
    @@ -81,7 +97,7 @@ Sub-agent prompts: self-contained, plain English. Include compressed global cont
    
     ### 4. Generate Per-Directory Maps
    
    -One `CODEMAP.md` per source directory (root + each subdirectory). Each map describes only the current directory level.
    +One `CODEMAP.md` per fully indexed source directory (root + included subdirectories). Excluded and boundary-only interiors do not require maps. Each map describes only the current directory level.
    
     ### 5. Assemble Root and Install Protocol
    
    @@ -113,6 +129,7 @@ Sections appear in this fixed order. Omit a section when it would be empty.
     | Section | Root | Mid-level | Leaf | Container |
     |---|---|---|---|---|
     | Summary | 1 sentence | 1 sentence | 1 sentence | 1 sentence |
    +| Scope and Exclusions | yes | — | — | root only |
     | Task Guide | yes | yes | yes | yes |
     | Subdirectories | yes | yes | — | yes |
     | Key Exports | yes | yes | if needed | — |
    @@ -120,6 +137,7 @@ Sections appear in this fixed order. Omit a section when it would be empty.
     | File Dependencies | — | immediate files only | yes | — |
    
     **Container directory** = directory with no source files, only subdirectories. Generate only Summary + Task Guide + Subdirectories.
    +Root containers also retain Scope and Exclusions, so deliberate omissions remain visible.
    
     ### Summary
    
    @@ -178,7 +196,7 @@ Rules:
     - One row per immediate file. **Never list files from child directories that have their own CODEMAP.**
     - `Domain` must match local Task Guide / Subdirectories values. `—` for trivial re-export files.
     - `Function`: one concise sentence. For large files, append `→ see <filename>.analysis.md`.
    -- Skip pure re-export `__init__.py` if exports are captured in Key Exports.
    +- Include pure re-export files such as `__init__.py` once in Files; Key Exports does not replace their canonical inventory row.
    
     ### File Dependencies
    
    @@ -292,10 +310,18 @@ The Deps column in Files tables marks cross-directory dependencies:
    
     ### Update Rules
    
    -After code changes, the agent autonomously evaluates:
    -- File/directory add, delete, move, rename → regenerate affected directory CODEMAP, update parent Subdirectories and Task Guide paths.
    -- Public symbol signature/return type change → update Key Exports and related Task Guide entries.
    -- Internal implementation change only → no update. Exception: update `.analysis.md` when Feature Index mapping becomes invalid.
    +Every code change requires evaluating all three operations within the affected task scope; perform each applicable operation without waiting for a separate cleanup request:
    +- Add: supply missing implementation or contract coverage and missing map entries for in-scope files/directories.
    +- Delete: remove confirmed redundant or superseded implementations and their exclusive tests, dependencies, build routes, documentation, and stale map/analysis entries. Check real consumers, supported platforms, dynamic/FFI entry points, and public contracts first.
    +- Modify: update existing implementations and canonical entries in place when responsibilities, interfaces, dependencies, paths, or analysis line ranges change. Moves/renames remove old routes and install current ones, including parent Task Guide/Subdirectories references.
    +
    +Completion requires a bijection between included source files/directories and their canonical local Files/Subdirectories entries: each entity appears exactly once, each entry resolves to a current entity, and excluded scope is explicit. Task Guide, Key Exports, and dependency links may reference the same entity; they must remain accurate without duplicating inventory ownership. Check source → map for missing/duplicate entries and map → source for stale entries and semantics, using the working tree rather than just the last commit.
    +
    +Deliberately unindexed areas must appear in root CODEMAP's Scope and Exclusions with root-relative paths/patterns, treatment (`Excluded` or `Boundary only`), and reason/basis. Boundary-only directories retain a marked directory entry, without an internal inventory obligation. Record rule sources for routine exclusions and explicit exceptions. When a task touches these areas or depends on their contracts, inspect the necessary code and reconsider scope; exclusion is neither deletion evidence nor a correctness exemption.
    +
    +Actively recheck retirement conditions for affected compatibility layers and migration code. Once replacement and consumer migration are verified, remove the old path in this task; retained exceptions need concrete consumers or a current contract and a removal condition. Do not accumulate `old/`, `v1/`, or `backup/` copies. Age or no local text references alone does not justify deletion.
    +
    +Pure internal equivalent changes may leave maps unchanged only after verifying that indexed responsibilities, contracts, dependencies, symbols, and line ranges remain accurate. Briefly report applicable additions/deletions/modifications, reasons for no action, and unresolved gaps; timestamps and passing builds alone do not establish map accuracy.
     ````
    
     ---
    # Python and editor artifacts
    __pycache__/
    *.py[cod]
    *.egg-info/
    .pytest_cache/
    .mypy_cache/
    .ruff_cache/
    .venv/
    venv/
    .idea/
    .vscode/
    *.swp
    *.swo
    
    # Build, coverage, and local maintenance state
    build/
    dist/
    out/
    coverage/
    .project-maintenance/
    
    # OS files
    .DS_Store
    Thumbs.db
    ✓ • 0ms

## Assistant

对照后确认，codemap 已经覆盖任务路由、按域过滤、大文件行段定位和地图增删改；本地版本目前没有配套脚本。重复建设这些能力的收益有限。

我会补两个实际缺口：给现有维护状态脚本加分页输出，保留完整变更计数；再把工具选择和动态脚本的使用条件写成按需读取的参考。另发现 codemap README 的部分规则落后于 SKILL.md，对照时以技能正文为准。

## Activity

    $ sed -n '1,130p' SKILL.md; sed -n '140,250p' scripts/maintenance_state.py; cat references/verification.md
    ---
    name: project-maintenance
    description: Keep a project's code maps, implementations, tests and build routes current after feature work, refactoring, API changes or backend migrations. Use for maintenance and retirement of superseded code in assembly, Rust, Python, TypeScript, C and C++ projects, especially native UI and dedicated inference engines.
    ---
    
    # Project Maintenance
    
    把维护作为当前开发任务的完成条件。同步受影响的地图，清除已被替代的实现和失效测试，验证仍然承诺的产品行为。对用户已经授权的任务内维护直接执行；跨出任务范围、改变支持平台或对外接口时，先给出具体影响。
    
    ## 确认当前契约
    
    - 从指定项目根目录开始，读取 `AGENTS.md`、当前需求及已有地图。保留规定的语言、渲染后端、推理引擎、平台支持和源码所有权边界。
    - 检查已提交、暂存、未暂存及新增文件。区分本次任务和其他人的未完成工作，不替别人整理不相关改动。
    - 新项目随首个可运行实现建立地图；已有项目以当前源码验证地图，不以旧文档推断不存在的接口。
    - 链接进 UI 仓库的引擎源码、子模块和独立 checkout 各自验证真实目录；一个仓库的记录不证明另一个源码树已经同步。
    
    Git 项目可先运行只读检查：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    ```
    
    脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    
    ## 随改动更新地图
    
    ### 双射性与增删改
    
    每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
    
    - **增**：补上当前契约所需的实现、有效验证和遗漏的地图条目。
    - **删**：删除已确认冗余或退役的代码，以及其专属测试、依赖、构建路线、文档和地图条目。
    - **改**：优先原地修正仍承担职责的实现，同步职责、接口、依赖、路径和行范围；不靠另加平行版本绕过旧问题。
    
    双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”。按 `$codemap` 格式，每个文件在本层 Files 中恰好一条，每个直接子目录在 Subdirectories 中恰好一条；其他格式明确等价的归属。纯重导出文件也不能因导出表存在而漏记。任务入口、符号表和依赖边允许多处指向同一实体，不构成重复归属；不要求每个函数或使用场景各有一张表。索引自身不递归作为源码索引。
    
    以当前工作树双向核对：源码到地图查缺漏和重复归属，地图到源码查幽灵路径和失真的职责、接口、依赖及行范围。明确排除范围，不能通过扩大忽略规则掩盖缺漏。完成时简要说明增删改的处理或不适用依据，以及仍未闭合的缺口。
    
    双射只约束声明的索引范围。根据用户关注点和项目约定，不关心的部分可以明确忽略或只保留目录边界，无需逐文件维护。必须在根地图的“范围与忽略项”中记录 `根目录相对路径/模式 | 处理方式 | 原因/依据`；处理方式为“排除”或“仅保留边界”。后者保留唯一目录条目并标注“内部未索引”，不要求内部文件表或子地图。常规忽略项可按默认规则、`.gitignore` 等明确来源归组，项目特有忽略项及重新纳入的例外必须明确列出。根地图是范围的权威记录，其他位置只引用。
    
    忽略索引不等于过时、可删除或免于正确性验证。后续任务触及忽略区或依赖其契约时，读取所需代码并重新评估范围；决定变化时同步标记，不能把未索引内容宣称为已核对。维护脚本的扫描排除与地图的索引排除分别服务于变更检测和导航，不能未经判断直接互相套用。
    
    使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    
    - 文件增删、移动、重命名：同步本层文件表、父目录路由及任务入口，移除已不存在目录的地图。
    - 接口或依赖变化：同步导出符号、调用入口、跨目录依赖及消费者。
    - 汇编与 FFI：记录真实符号、调用约定、结构布局和偏移、内存所有权、回调线程，以及 C/Rust 包装层；用头文件、声明、调用点和链接符号核对。
    - UI 与推理引擎：记录事件到状态到渲染/推理的路径；模型边界写明 tensor shape、dtype、布局、量化格式和必要的转换。
    - 实现变化：若职责、数据流、契约或大文件功能行范围变化，仍然更新相关说明；纯内部等价修改可以不改地图，并说明判断依据。
    
    地图保留一份当前版本，原地更新。只在 agent 规则文件中放简短入口和完成条件，避免复制整份地图。
    
    ## 文档一起更新和退役
    
    - 代码、接口、命令、目录、配置、支持平台或用户流程变化时，检查 README、架构说明、运行手册、示例、任务入口和交叉链接；把仍然有效的说明改成当前行为。
    - 功能、接口或构建路线退役时，删除只服务于它的文档、示例、截图、fixture、导航条目和失效链接；先核对反向链接与仍在使用的共享资源。Git 历史承担已删除内容的留存。
    - 保留一份权威说明，合并重复文档，避免用过时的 `old/`、`v1/`、`backup/` 文档树掩盖当前状态。文档删除必须有当前契约或调用关系依据，不能只因文件很旧或无人引用。
    - 完成维护后，至少检查受影响 Markdown 链接、命令、路径和状态描述；不能验证的外部链接或平台行为要明确记录。
    
    ## 实现与测试一起退役
    
    **时效淘汰性**：每次变更主动复查受影响旧实现和临时层的保留依据、迁移状态与移除条件。替代实现已验证、消费者已迁移或旧需求已撤销时，在本次任务内完成退役；不能仅在地图上标成 deprecated 后无限保留。暂时仍需保留的分支必须记录具体消费者或当前契约及可检查的移除条件，在后续相关改动时重新核对。时效由当前需求和迁移证据判定，不按文件年龄或任意期限机械删除。
    
    对候选项先核对用途、调用者、构建目标、平台和公开契约，再在本次授权范围内处理：
    
    - 已替换并完成迁移的实现：清除旧代码、专属包装层、构建入口、依赖、文档、测试与 fixture；历史由 Git 保留，不复制 `old/`、`v1/`、`backup/` 源码树。
    - 临时适配层：迁移完成就移除；仍有使用者时，写清当前消费者和具体移除条件。
    - 重复或仅验证 mock 自身的测试：合并或删除，保留不同输入、平台、边界条件和已发现故障的有效断言。
    - 测试取消或改变的需求：按当前契约退役或重写，同时核对相关 fixture、runner 配置和文档。
    - 失败、被跳过、年代久远、覆盖率低或图中没有入边，都不是单独的删除依据。先区分产品回归、环境问题、有效平台测试和真正失效的需求。
    - 新测试只验证本次行为、接口或实际故障，避免把当前实现细节再写一遍。对仍有效的回归保持独立检测能力；不能为使检查通过而删除断言、改宽误差或重录错误的快照。
    
    外部导出、动态注册、函数指针、FFI、汇编入口和跨平台分支需要额外核对消费者。只证明“无本地文本引用”时，报告剩余不确定性，继续完成其余可确认的维护。
    
    ## 验证并记录当前状态
    
    按 [验证与退役判据](references/verification.md) 只选择本次涉及的语言、平台、UI 或推理验证。优先复用现有验证入口；不要为了使用这个 skill 重建一套重复测试框架。
    
    完成本次维护和有效验证后，在原地替换单份审查记录：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py record --root <project-root> --reviewed
    python3 <skill-dir>/scripts/maintenance_state.py check --root <project-root>
    ```
    
    `--reviewed` 表示 agent 已实际审查当前范围并完成应做的验证，不是用户批准，也不能替代验证。未完成的关联改动、未知失败或未验证的其他源码树必须如实报告；不能为让检查变绿而重录。
    
    记录位于 `.project-maintenance/state.json`，包含当前文件哈希与审查版本。`check` 在文件或审查范围变化后返回非零，供已有验证命令、pre-commit 或 CI 调用；新增这些集成时使用项目现有方式，编辑一个可重复更新的入口。脚本的范围、排除模式和能力限制见验证参考。
    
    最终报告本次同步的地图、退役项及其依据、实际验证结果和未解决问题。指标是当前契约和可维护性，不是删除数量、测试数量或通过率。
    def write_state(root: Path, state: dict) -> None:
        directory = root / STATE_PATH.parent
        if directory.is_symlink():
            raise ValueError("Maintenance state directory must not be a symlink")
        directory.mkdir(exist_ok=True)
        temporary = None
        try:
            with tempfile.NamedTemporaryFile(
                mode="w", encoding="utf-8", dir=directory, prefix=".state-", delete=False
            ) as stream:
                temporary = Path(stream.name)
                json.dump(state, stream, indent=2, sort_keys=True)
                stream.write("\n")
            os.replace(temporary, directory / STATE_PATH.name)
        finally:
            if temporary is not None and temporary.exists():
                temporary.unlink()
    
    
    def print_report(report: dict, as_json: bool) -> None:
        if as_json:
            print(json.dumps(report, indent=2, sort_keys=True))
            return
        print(f"Project: {report['root']}")
        print(f"Status: {report['status']}")
        for group in ("files", "maps"):
            print(f"{group}: {report['counts'][group]} monitored")
            for kind, paths in report["changes"][group].items():
                if paths:
                    print(f"  {kind}: {len(paths)}")
                    for name in paths[:40]:
                        print("    " + ascii(name))
                    if len(paths) > 40:
                        print("    ... use --json for the complete list")
        if report["scope_changed"]:
            print("Scope changed; review exclusions before recording")
        if report["separate_checkouts"]:
            print("Linked/submodule content requires separate review:")
            for checkout in report["separate_checkouts"]:
                print("  " + ascii(checkout["path"]) + " (" + checkout["kind"] + ")")
    
    
    def main() -> int:
        parser = argparse.ArgumentParser(description=__doc__)
        parser.add_argument("action", choices=("inspect", "record", "check"))
        parser.add_argument("--root", required=True, help="Path inside the intended Git project")
        parser.add_argument("--json", action="store_true", help="Emit complete structured output")
        parser.add_argument("--reviewed", action="store_true", help="Record only after actual review")
        parser.add_argument("--exclude", action="append", help="Replace custom root-relative exclusions")
        parser.add_argument("--clear-excludes", action="store_true")
        parser.add_argument("--include-generated", action="store_true", default=None)
        parser.add_argument("--exclude-generated", dest="include_generated", action="store_false", default=None)
        args = parser.parse_args()
        if args.action == "record" and not args.reviewed:
            parser.error("record requires --reviewed after maintenance and verification")
        if args.reviewed and args.action != "record":
            parser.error("--reviewed applies only to record")
        if args.exclude is not None and args.clear_excludes:
            parser.error("Use --exclude or --clear-excludes, not both")
        try:
            root = repo_root(args.root)
            old = load_state(root)
            patterns = sorted(set(args.exclude)) if args.exclude is not None else (
                [] if args.clear_excludes else (old or {}).get("excludes", [])
            )
            if any(not p or p.startswith("/") or ".." in Path(p).parts for p in patterns):
                raise ValueError("Exclusions must be nonempty root-relative patterns without '..'")
            include_generated = args.include_generated if args.include_generated is not None else (
                (old or {}).get("include_generated", False)
            )
            current = inventory(root, patterns, include_generated)
            changes = {group: differences((old or {}).get(group, {}), current[group])
                       for group in ("files", "maps")}
            scope_changed = old is not None and (
                patterns != old["excludes"] or include_generated != old["include_generated"]
            )
            needs_review = old is None or scope_changed or any(
                paths for group in changes.values() for paths in group.values()
            )
            status = "review-required" if needs_review else "matches-reviewed-state"
            if args.action == "record":
                state = {
                    "schema_version": SCHEMA_VERSION,
                    "reviewed_at": datetime.now(timezone.utc).isoformat(),
                    "head": git(root, "rev-parse", "--verify", "HEAD", optional=True).decode().strip() or None,
                    "excludes": patterns, "include_generated": include_generated, **current,
                }
                write_state(root, state)
                status = "recorded-reviewed-state"
            print_report({
                "root": str(root), "status": status, "scope_changed": scope_changed,
                "counts": {group: len(current[group]) for group in ("files", "maps")},
                "changes": changes, "separate_checkouts": current["separate_checkouts"],
                "excludes": patterns, "include_generated": include_generated,
            }, args.json)
            return int(needs_review) if args.action == "check" else 0
        except (OSError, ValueError, UnicodeError) as error:
            if args.json:
                print(json.dumps({"error": str(error)}), file=sys.stderr)
            else:
                print(f"Error: {error}", file=sys.stderr)
            return 2
    
    
    if __name__ == "__main__":
        sys.exit(main())
    # 验证与退役判据
    
    只读取本次涉及的部分。使用项目已有命令、锁定版本、功能组合和支持平台，先记录已存在的失败，再核对本次差异。缺少完成验证所必需的工具或数据时补齐并执行；不能把未验证写成通过。
    
    ## 语言与接口边界
    
    | 场景 | 需要的证据 |
    | --- | --- |
    | 汇编、C、C++ | 用实际目标和编译选项构建并链接；核对导出/引用符号、ABI、栈与缓冲区对齐、寄存器保存、布局断言及有效的平台分支。删汇编符号前核对 C/Rust 声明、链接脚本、动态查找和函数指针注册。 |
    | Rust / FFI | 当前 feature 与 target 下的构建和接口测试；对应安全契约、布局、所有权及异常边界。Miri 可验证 Rust 侧契约，但不能证明任意 C/汇编实际调用正确。 |
    | Python | 当前环境的类型、lint 和相关功能验证；核对动态导入、插件注册、CLI、模型导出/转换入口及包的公开接口。 |
    | TypeScript | 当前配置的类型检查、构建与相关行为测试；核对动态加载、路由、注册表、包导出及原生桥接。死代码工具只提供候选。 |
    
    ## 原生与 Web UI
    
    - 原生 UI：运行实际窗口，验证此次涉及的输入、焦点、文本、缩放、布局、渲染路径及与真实引擎的交互；检查截图/帧与运行日志。构建成功和生成一张图不证明交互完整。
    - Web UI：在真实浏览器中验证此次流程、控制台、网络和相应视口；使用当前页面状态定位元素。纯 DOM 或组件快照不证明连接实际后端后的流程。
    - 视觉基线只在需求确认改变且实际画面核对后更新。移除弃用界面的快照与 fixture 时，同时核对共用资源的消费者。
    
    ## 专用推理框架
    
    - 从原始模型或已验证的参考实现建立输入与输出契约：shape、dtype、stride/layout、量化参数、采样/帧约定以及适用精度。
    - 按实际工作负载检查数值对齐和完整推理路径。新内核需要验证尾部尺寸、对齐、有效 ISA 分派和 fallback；“更快”不证明“相同”。
    - 性能采用同机、同负载、同契约的稳定比较，记录端到端与真实热点。复用已有 `performance-gradient-optimization` 或项目等效方法。
    - 保留当前对齐/性能基线的明确用途；淘汰无用的候选实现、重复基准入口和失效数据。基线输入、期望结果与硬件/模型来源需要可追溯，历史结果交由项目既有存档机制保存。
    
    ## 持续维护检查脚本
    
    `scripts/maintenance_state.py` 仅依赖 Python 标准库和 Git，支持包括尚无首次提交的 Git 工作树：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py inspect --root . --json
    python3 <skill-dir>/scripts/maintenance_state.py record --root . --reviewed
    python3 <skill-dir>/scripts/maintenance_state.py check --root .
    ```
    
    - 扫描 Git 跟踪文件及未被忽略的新增文件，比较实际内容哈希；检测修改、增删、重命名及地图本身的变化。模式或时间戳相同的内容修改仍会触发。
    - tracked fixture、lockfile、配置与文档也在范围内。`CODEMAP.md`、`codemap.md` 和 `*.analysis.md` 在报告中单列；记录文件自身不进入指纹。
    - 默认排除缓存/依赖目录，以及根目录的 `build/`、`dist/`、`out/`、`coverage/`。若项目把真实源码放在这些位置，通过 `--include-generated` 纳入；该选择会记录在范围中。
    - 大模型、数据集或受项目明确排除的产物，可使用根目录相对的 `--exclude 'weights/**'` 等模式；目录模式如 `datasets/` 排除其子树。模式影响扫描范围，不能拿它掩盖未完成维护。设置会保存在审查记录中，后续检查复用；显式重新提供模式会替换已保存的自定义排除。
    - 例如：`inspect --root . --exclude 'weights/**'`；确认范围后以相同选项运行 `record --reviewed`。恢复自定义排除为空使用 `--clear-excludes`。修改排除范围会要求新的审查记录。
    - 符号链接只记录链接目标，不跟随读取外部文件；子模块只记录 checkout 状态。报告中的 `separate_checkouts` 必须在其真实源码树单独审查和检查，链接本身不变不代表引擎内容不变。
    - `record` 只原子替换 `.project-maintenance/state.json`。若使用 CI，可把这份记录交给 Git；后续在同一路径更新，不生成每次任务的副本。
    - `inspect` 只读并返回清单。`check` 返回 `0` 表示该范围与已审查指纹一致，`1` 表示需审查或尚无记录，`2` 表示仓库、读取或记录错误；错误不能当作干净状态。
    - 一致的指纹只证明“之后没有新的受监测文件变化”，不证明地图语义、死代码判断或产品正确性。实际调用者和有意义的验证仍由 agent 核对。
    
    在已有项目中接入检查脚本时使用该 skill 的实际路径或项目自己的固定入口。不要假定其他机器安装在同一个 home 路径，也不要把 skill 私有源码大量复制进项目。
    ✓ • 0ms

## Activity

    Opened https://ast-grep.github.io/reference/cli.md

## Activity

    file changes: Completed · 4 changes
    Update { move_path: None }: /home/fuurin/code/skills/project-maintenance/SKILL.md
    @@ -19,6 +19,12 @@
     ```bash
    -python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    +python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root> --json --max-paths 20
     ```
     
    -脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    +脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。限量输出仍保留完整计数和审查状态；分页或先只取计数的方式见[验证参考](references/verification.md)。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    +
    +## 工具与 agent 协作
    +
    +用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
    +
    +先取计数、路径和行范围，再读必要内容；完整扫描结果可以留在工具侧。按任务需要选择 `rg`、AST、语言服务器或动态脚本，复用工具及维护状态入口。需要选型、批量核对或自建脚本时，读取[自动化与 codemap 协作](references/automation.md)。
     
    @@ -34,3 +40,3 @@
     
    -双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”。按 `$codemap` 格式，每个文件在本层 Files 中恰好一条，每个直接子目录在 Subdirectories 中恰好一条；其他格式明确等价的归属。纯重导出文件也不能因导出表存在而漏记。任务入口、符号表和依赖边允许多处指向同一实体，不构成重复归属；不要求每个函数或使用场景各有一张表。索引自身不递归作为源码索引。
    +双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”，每个实体恰好归属一条；重导出文件也需纳入。任务入口、符号表和依赖边允许多处指向同一实体。采用 `$codemap` 时沿用其本层 Files/Subdirectories、父子路由及索引产物排除规则；其他地图保留等价归属，使用已有格式。
     
    @@ -38,3 +44,3 @@
     
    -双射只约束声明的索引范围。根据用户关注点和项目约定，不关心的部分可以明确忽略或只保留目录边界，无需逐文件维护。必须在根地图的“范围与忽略项”中记录 `根目录相对路径/模式 | 处理方式 | 原因/依据`；处理方式为“排除”或“仅保留边界”。后者保留唯一目录条目并标注“内部未索引”，不要求内部文件表或子地图。常规忽略项可按默认规则、`.gitignore` 等明确来源归组，项目特有忽略项及重新纳入的例外必须明确列出。根地图是范围的权威记录，其他位置只引用。
    +双射只约束声明的索引范围。根地图统一记录排除项、仅保留边界的目录、原因及纳入例外；内部未索引的目录只保留唯一边界条目。采用 `$codemap` 时沿用其 Scope and Exclusions 格式，其他位置引用根地图的范围。
     
    Add: /home/fuurin/code/skills/project-maintenance/references/automation.md
    # 自动化与 codemap 协作
    
    需要选择工具、批量核对或动态生成辅助脚本时读本参考。工具提取可重复的事实，agent 判断当前契约与变更影响；读取范围随证据扩展。本文描述两种技能的协作，地图格式与导航规则由可用的 `$codemap` 或项目现有地图维护。
    
    ## 已有能力及归属
    
    | 能力 | 复用入口 | project-maintenance 的工作 |
    | --- | --- | --- |
    | 任务路由、Domain 过滤、条件读取消费者 | codemap 的 Task Guide、Files、Dependencies | 用变更证据选择入口，判断公开契约是否受影响 |
    | 大文件功能到行范围的定位 | codemap 的 `.analysis.md` Feature Index | 核对受影响范围仍对应当前源码 |
    | 文件/目录唯一归属、范围和排除项 | codemap 的 Files、Subdirectories、根 Scope and Exclusions | 检查当前工作树的缺漏、重复、幽灵路径并同步 |
    | 地图增删改与父子路由更新 | codemap 维护模式 | 将地图更新纳入实现、文档、构建和验证的完成条件 |
    | 审查之后发生了哪些变化 | `maintenance_state.py` 的 inspect/check | 结合 Git 差异区分本次任务和其他改动 |
    | 旧实现、测试和迁移层是否应退役 | 当前源码、消费者、产品契约及验证入口 | agent 判断保留依据与移除条件，执行获授权的维护 |
    
    维护状态的扫描范围与地图的索引范围分别保存在审查记录和根地图中；扫描会包含许多无需索引的配置、fixture 和文档。临时候选清单用于本次操作，不成为另一份权威地图。采用 codemap 时使用其维护模式；普通维护沿用既有配置，只有新建或改变索引范围时才处理相应配置问题。
    
    ## 按需求选择工具
    
    | 要解决的问题 | 工具与使用条件 | 返回给 agent 的信息 |
    | --- | --- | --- |
    | 找文件及本次差异 | Git、`rg --files`；审查基线用已有状态脚本 | 状态、数量、路径、差异片段 |
    | 找名称、路径和文本消费者 | `rg -l` 先收候选，`rg -n -F` 再定位；按目录/类型缩小范围 | 去重路径及必要的匹配行 |
    | 找语法结构、批量迁移同类调用 | 有需要时使用 ast-grep 的结构搜索、outline、重写预览 | 符号/节点、文件、范围、替换差异 |
    | 分辨同名符号、查询引用、语义重命名 | 项目已有的语言服务器，例如 clangd、rust-analyzer；配置、feature 和索引覆盖需有效 | 真实符号的定义与引用位置 |
    | 跨语言声明定位，缺少现成结构提取器 | 已安装的 Universal Ctags；Python 可直接用标准库 AST | 符号及行号，作为导航候选 |
    | 地图表格与当前目录的批量集合核对 | 先复用 codemap/项目工具；缺少时生成限定格式的临时比较脚本 | 缺漏、重复归属、幽灵路径、越界行范围 |
    | 重复全量提取确实耗时 | 优先复用语言服务器缓存；有测量依据再考虑 SQLite 存放按内容哈希更新的事实 | 与本次变更相关的行，标明来源和有效版本 |
    
    ast-grep 支持结构搜索、JSON 输出和重写；当前官方版本还提供 outline。使用前检查本地 `--help` 的能力，缺少工具时先选已安装且足够的方式。[官方 CLI](https://ast-grep.github.io/reference/cli.html) 与[工具指南](https://ast-grep.github.io/guide/tooling-overview.html)。Universal Ctags 的 JSON 支持取决于编译选项，其符号清单用于定位。[官方 JSON 文档](https://docs.ctags.io/en/latest/man/ctags-json-output.5.html)。clangd 的后台索引提供项目引用信息，当前打开文件的动态索引用于跟随编辑变化。[索引设计](https://clangd.llvm.org/design/indexing)。
    
    结构匹配和文本命中都只是证据。无命中时核对搜索忽略项、动态注册、FFI、导出接口和有效平台分支；单凭它们不执行删除。语言服务器也只覆盖其有效配置下的源码。源码内容变化后，旧行号、片段和提取结果需要重新核对。
    
    ## 一次维护怎样执行
    
    1. **取增量事实**：用 `inspect --json --max-paths 20` 取得完整计数和首批路径，必要时分页。首次没有审查记录时，清单表示待建立基线，不能把全部 added 视为本次新增；结合 Git 的暂存、未暂存差异与未跟踪文件判断。
    2. **通过地图定向**：沿 codemap 的任务路由进入受影响目录，用 Feature Index 确定大文件行范围。有具体契约缺口再查消费者，消费者还可能位于本次未修改的目录。
    3. **取得最小证据**：批量运行独立查询，工具侧去重和筛选；返回路径、符号、行号和所需片段。完整清单较大时保存在临时文件并明示截断，读取后续项或按任务筛选，不能用一个 `head` 隐藏未知范围。
    4. **agent 判断并执行增删改**：以实现、消费者和当前契约决定操作。适用的统一替换先看匹配与差异预览，再执行受限的批量编辑；任意字符串替换无法代替语义重命名。
    5. **闭合影响**：同步 codemap、相关说明、构建路线和有效验证。完成实际审查后才 record，再 check；分页参数只压缩报告，不缩小记录范围。
    
    例如先用 `rg -l -F -e 'OldEngine' src tests` 取得候选路径；再对选定文件定位行段。具体命中需要 agent 判断，不能把测试字符串、注释和实际调用混为一个消费者。Git 与 rg 的文件过滤结果也不能直接充当地图的声明范围。
    
    ## 动态自建工具的条件
    
    当既有命令的组合已经够用时直接组合。有明确的重复计算或格式转换需求时，再写一个窄小辅助脚本。例如 codemap 格式的 Files/Subdirectories 双向集合比较应复用或补到 codemap 的工具归属；维护状态继续使用本技能的单一脚本入口。
    
    临时脚本放在获允许的临时目录或项目既有本地工具目录。参数明确给出根目录、输入、声明的范围、格式和输出位置；未知表格格式要报“未解析”，不能假装核对成功。只比较权威清单条目，避免把 Task Guide、Key Exports 和导航链接算作重复归属。范围、例外和仅保留边界需依据根地图解释，不能照搬维护扫描的排除项。
    
    把事实与候选判断分开输出，保留来源路径、当前内容标识和必要行号；输出包括完整计数、返回数量、遗漏/截断、后续读取入口和错误。解析、读取或子进程失败时返回失败；参数化调用工具，避免将仓库文本拼成可执行的 shell 命令。批量写入使用具体编辑清单或补丁，并核对预期内容仍匹配当前文件。
    
    只用于一次任务的脚本无需长期维护。经实际复用证明有收益的脚本再归入对应 skill/项目工具，并补充使用说明和行为验证；保留一套接口。节省 token 主要靠减少返回的无关内容和重复读取；多 agent 会增加总调用量，只有已获授权且任务能独立分工时，才用它隔离中间上下文。
    Update { move_path: None }: /home/fuurin/code/skills/project-maintenance/references/verification.md
    @@ -36,2 +36,8 @@
     
    +agent 首次读取可用 `inspect --root . --json --max-paths 20`；只取计数用 `--max-paths 0`。限量 JSON 的 `change_counts` 和 `separate_checkout_count` 始终覆盖完整扫描，`changes` 与 `separate_checkouts` 只包含本页。分页顺序为地图的增/改/删、其他文件的增/改/删、独立 checkout，各组按路径排序。
    +
    +按 `pagination.next_offset` 继续传入 `--offset`，并核对各页 `report_id` 一致；内容、审查记录或范围变化时重新取页。`pagination.total/returned/omitted/truncated` 明示完整量与本页量；计数模式不提供下一页，应改用正数限量。分页限制路径条目数量，路径长度和范围元数据仍会影响输出大小。省略 `--max-paths` 恢复原有完整 JSON；文本输出仍使用原有摘要。
    +
    +输出限量不改变扫描范围、审查记录或 `check` 退出码。不要把本页空列表、最后一页或截断结果当成无变更；以完整状态、计数和实际审查为准。`--root` 会解析为 Git 顶层，单个包的任务应再按地图与任务路径筛选，不能把仓库级记录说成仅审查了当前包。
    +
     - 扫描 Git 跟踪文件及未被忽略的新增文件，比较实际内容哈希；检测修改、增删、重命名及地图本身的变化。模式或时间戳相同的内容修改仍会触发。
    Update { move_path: None }: /home/fuurin/code/skills/project-maintenance/scripts/maintenance_state.py
    @@ -158,2 +158,42 @@
     
    +def paginate_report(report: dict, max_paths: int, offset: int) -> dict:
    +    # Keep full counts and review status; pagination is presentation, not scope.
    +    entries = [
    +        (group, kind, name)
    +        for group in ("maps", "files")
    +        for kind, names in report["changes"][group].items()
    +        for name in names
    +    ]
    +    entries.extend(("separate_checkouts", "", item)
    +                   for item in report["separate_checkouts"])
    +    page = entries[offset:offset + max_paths]
    +    limited = {
    +        **report,
    +        "change_counts": {
    +            group: {kind: len(names) for kind, names in changes.items()}
    +            for group, changes in report["changes"].items()
    +        },
    +        "separate_checkout_count": len(report["separate_checkouts"]),
    +        "changes": {
    +            group: {kind: [] for kind in changes}
    +            for group, changes in report["changes"].items()
    +        },
    +        "separate_checkouts": [],
    +        "pagination": {
    +            "offset": offset, "limit": max_paths, "total": len(entries),
    +            "returned": len(page), "omitted": len(entries) - len(page),
    +            "truncated": len(page) < len(entries),
    +            "next_offset": offset + len(page) if page and (
    +                offset + len(page) < len(entries)
    +            ) else None,
    +        },
    +    }
    +    for group, kind, item in page:
    +        if group == "separate_checkouts":
    +            limited[group].append(item)
    +        else:
    +            limited["changes"][group][kind].append(item)
    +    return limited
    +
    +
     def print_report(report: dict, as_json: bool) -> None:
    @@ -185,3 +225,5 @@
         parser.add_argument("--root", required=True, help="Path inside the intended Git project")
    -    parser.add_argument("--json", action="store_true", help="Emit complete structured output")
    +    parser.add_argument("--json", action="store_true", help="Emit structured output; complete unless --max-paths is set")
    +    parser.add_argument("--max-paths", type=int, help="Limit total JSON change/checkout entries; 0 emits counts only")
    +    parser.add_argument("--offset", type=int, default=0, help="Skip entries when using --max-paths")
         parser.add_argument("--reviewed", action="store_true", help="Record only after actual review")
    @@ -198,2 +240,6 @@
             parser.error("Use --exclude or --clear-excludes, not both")
    +    if args.max_paths is not None and (not args.json or args.max_paths < 0):
    +        parser.error("--max-paths requires --json and a nonnegative limit")
    +    if args.offset < 0 or (args.offset and args.max_paths is None):
    +        parser.error("--offset requires --max-paths and a nonnegative offset")
         try:
    @@ -228,3 +274,3 @@
                 status = "recorded-reviewed-state"
    -        print_report({
    +        report = {
                 "root": str(root), "status": status, "scope_changed": scope_changed,
    @@ -233,3 +279,13 @@
                 "excludes": patterns, "include_generated": include_generated,
    -        }, args.json)
    +        }
    +        if args.max_paths is not None:
    +            snapshot = {
    +                "current": current, "reviewed": old,
    +                "excludes": patterns, "include_generated": include_generated,
    +            }
    +            report["report_id"] = hashlib.sha256(
    +                json.dumps(snapshot, sort_keys=True).encode()
    +            ).hexdigest()
    +            report = paginate_report(report, args.max_paths, args.offset)
    +        print_report(report, args.json)
             return int(needs_review) if args.action == "check" else 0

## Assistant

协作规则已写入技能参考，状态脚本也加了分页。地图导航和格式继续沿用 codemap，临时清单只服务本次操作。分页会保留完整计数、遗漏数量和报告标识，方便发现读取多页期间的文件变化；接下来验证限量输出不会把待审查状态误报为完成。

## Activity

    $ python3 /home/fuurin/.codex/skills/.system/skill-creator/scripts/quick_validate.py /home/fuurin/code/skills/project-maintenance; git diff --check; git diff --stat; python3 scripts/maintenance_state.py inspect --root . --json --max-paths 5
    Skill is valid!
     codemap/SKILL.md                                 | 38 ++++++++++++---
     project-maintenance/SKILL.md                     | 28 ++++++++++-
     project-maintenance/references/verification.md   |  6 +++
     project-maintenance/scripts/maintenance_state.py | 62 ++++++++++++++++++++++--
     4 files changed, 123 insertions(+), 11 deletions(-)
    {
      "change_counts": {
        "files": {
          "added": 71,
          "modified": 0,
          "removed": 0
        },
        "maps": {
          "added": 0,
          "modified": 0,
          "removed": 0
        }
      },
      "changes": {
        "files": {
          "added": [
            ".gitignore",
            "README.md",
            "codemap/LICENSE",
            "codemap/README.md",
            "codemap/README_EN.md"
          ],
          "modified": [],
          "removed": []
        },
        "maps": {
          "added": [],
          "modified": [],
          "removed": []
        }
      },
      "counts": {
        "files": 71,
        "maps": 0
      },
      "excludes": [],
      "include_generated": false,
      "pagination": {
        "limit": 5,
        "next_offset": 5,
        "offset": 0,
        "omitted": 66,
        "returned": 5,
        "total": 71,
        "truncated": true
      },
      "report_id": "ed60bc12ad5f1ee9e369533b5afa671b49ac46c433ecb6f0f9b09bba371be8e7",
      "root": "/home/fuurin/code/skills",
      "scope_changed": false,
      "separate_checkout_count": 0,
      "separate_checkouts": [],
      "status": "review-required"
    }
    ✓ • 75ms

## Activity

    $ python3 - <<'PY'
        import json
        import os
        from pathlib import Path
        import subprocess
        import tempfile
        
        script = Path('scripts/maintenance_state.py').resolve()
        
        def run(root, action='inspect', *options, expected=0):
            result = subprocess.run(
                ['python3', str(script), action, '--root', str(root), '--json', *options],
                capture_output=True, text=True,
            )
            assert result.returncode == expected, (result.returncode, result.stderr, result.stdout)
            return json.loads(result.stdout) if result.stdout else None
        
        with tempfile.TemporaryDirectory(prefix='maintenance-pagination-') as directory:
            root = Path(directory)
            subprocess.run(['git', 'init', '-q', str(root)], check=True)
            (root / 'a.py').write_text('value = 1\n')
            (root / 'b.py').write_text('value = 2\n')
            (root / 'CODEMAP.md').write_text('fixture map\n')
            (root / 'linked-engine').symlink_to('/tmp/not-a-real-engine')
            initial = run(root, 'check', '--max-paths', '0', expected=1)
            assert initial['status'] == 'review-required'
            assert initial['change_counts']['files']['added'] == 3
            assert initial['change_counts']['maps']['added'] == 1
            assert initial['separate_checkout_count'] == 1
            assert initial['pagination']['total'] == 5 and initial['pagination']['returned'] == 0
            first = run(root, 'inspect', '--max-paths', '1')
            assert first['changes']['maps']['added'] == ['CODEMAP.md']
            assert first['pagination']['next_offset'] == 1
            run(root, 'record', '--reviewed', '--max-paths', '0')
            stored = json.loads((root / '.project-maintenance/state.json').read_text())
            assert len(stored['files']) == 3 and len(stored['maps']) == 1
            clean = run(root, 'check', '--max-paths', '0')
            assert clean['status'] == 'matches-reviewed-state'
        
            before = (root / 'a.py').stat()
            (root / 'a.py').write_text('value = 9\n')
            os.utime(root / 'a.py', ns=(before.st_atime_ns, before.st_mtime_ns))
            (root / 'b.py').unlink()
            unusual_name = 'new file\nwith newline.py'
            (root / unusual_name).write_text('value = 3\n')
            (root / 'CODEMAP.md').write_text('changed fixture map\n')
            full = run(root)
            assert 'pagination' not in full and 'report_id' not in full
            assert full['changes']['files'] == {
                'added': [unusual_name], 'modified': ['a.py'], 'removed': ['b.py'],
            }
            rebuilt = {group: {kind: [] for kind in full['changes'][group]} for group in ('files', 'maps')}
            checkouts = []
            offset = 0
            report_id = None
            page_count = 0
            while True:
                page = run(root, 'check', '--max-paths', '2', '--offset', str(offset), expected=1)
                assert page['status'] == 'review-required'
                assert page['pagination']['returned'] <= 2
                report_id = report_id or page['report_id']
                assert page['report_id'] == report_id
                assert page['change_counts']['files'] == {'added': 1, 'modified': 1, 'removed': 1}
                for group in rebuilt:
                    for kind in rebuilt[group]:
                        rebuilt[group][kind].extend(page['changes'][group][kind])
                checkouts.extend(page['separate_checkouts'])
                page_count += 1
                offset = page['pagination']['next_offset']
                if offset is None:
                    break
            assert rebuilt == full['changes'] and checkouts == full['separate_checkouts']
            empty = run(root, 'check', '--max-paths', '1', '--offset', '999', expected=1)
            assert empty['pagination']['returned'] == 0 and empty['pagination']['truncated']
            counts = run(root, 'check', '--max-paths', '0', expected=1)
            assert counts['pagination']['total'] == 5 and counts['pagination']['next_offset'] is None
            (root / 'a.py').write_text('value = 8\n')
            changed = run(root, 'inspect', '--max-paths', '2')
            assert changed['report_id'] != report_id
            scope = run(root, 'check', '--max-paths', '0', '--exclude', 'a.py', expected=1)
            assert scope['scope_changed'] and scope['status'] == 'review-required'
            run(root, 'inspect', '--max-paths', '-1', expected=2)
            run(root, 'inspect', '--offset', '1', expected=2)
            run(root / 'missing', 'inspect', '--max-paths', '0', expected=2)
            print(json.dumps({
                'result': 'passed', 'pages': page_count,
                'checked': ['unborn Git repository', 'counts and checkout visibility',
                            'full record under output limit', 'dirty and scope-change exit codes',
                            'pagination reconstruction', 'stable and invalidated report ID',
                            'same-size same-mtime edits', 'unusual filenames', 'invalid arguments and root'],
            }, ensure_ascii=False))
        
        root = Path.cwd()
        full_result = subprocess.run(['python3', str(script), 'inspect', '--root', str(root), '--json'], capture_output=True, text=True, check=True)
        limited_result = subprocess.run(['python3', str(script), 'inspect', '--root', str(root), '--json', '--max-paths', '20'], capture_output=True, text=True, check=True)
        full = json.loads(full_result.stdout)
        limited = json.loads(limited_result.stdout)
        print(json.dumps({'local_output_chars': {'full': len(full_result.stdout), 'page_20': len(limited_result.stdout)}, 'same_status': full['status'] == limited['status'], 'complete_counts': limited['change_counts'], 'pagination': limited['pagination']}))
        PY
    {"result": "passed", "pages": 3, "checked": ["unborn Git repository", "counts and checkout visibility", "full record under output limit", "dirty and scope-change exit codes", "pagination reconstruction", "stable and invalidated report ID", "same-size same-mtime edits", "unusual filenames", "invalid arguments and root"]}
    {"local_output_chars": {"full": 4340, "page_20": 1857}, "same_status": true, "complete_counts": {"files": {"added": 71, "modified": 0, "removed": 0}, "maps": {"added": 0, "modified": 0, "removed": 0}}, "pagination": {"limit": 20, "next_offset": 20, "offset": 0, "omitted": 51, "returned": 20, "total": 71, "truncated": true}}
    ✓ • 1.35s

## Activity

    $ git diff -- project-maintenance/SKILL.md project-maintenance/references/verification.md project-maintenance/scripts/maintenance_state.py; git status --short; wc -l references/automation.md; nl -ba SKILL.md | sed -n '15,65p'; nl -ba scripts/maintenance_state.py | sed -n '155,220p'
    diff --git a/project-maintenance/SKILL.md b/project-maintenance/SKILL.md
    index 8cc02dc..f5f1c4d 100644
    --- a/project-maintenance/SKILL.md
    +++ b/project-maintenance/SKILL.md
    @@ -17,13 +17,35 @@ description: Keep a project's code maps, implementations, tests and build routes
     Git 项目可先运行只读检查：
    
     ```bash
    -python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    +python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root> --json --max-paths 20
     ```
    
    -脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    +脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。限量输出仍保留完整计数和审查状态；分页或先只取计数的方式见[验证参考](references/verification.md)。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    +
    +## 工具与 agent 协作
    +
    +用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
    +
    +先取计数、路径和行范围，再读必要内容；完整扫描结果可以留在工具侧。按任务需要选择 `rg`、AST、语言服务器或动态脚本，复用工具及维护状态入口。需要选型、批量核对或自建脚本时，读取[自动化与 codemap 协作](references/automation.md)。
    
     ## 随改动更新地图
    
    +### 双射性与增删改
    +
    +每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
    +
    +- **增**：补上当前契约所需的实现、有效验证和遗漏的地图条目。
    +- **删**：删除已确认冗余或退役的代码，以及其专属测试、依赖、构建路线、文档和地图条目。
    +- **改**：优先原地修正仍承担职责的实现，同步职责、接口、依赖、路径和行范围；不靠另加平行版本绕过旧问题。
    +
    +双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”，每个实体恰好归属一条；重导出文件也需纳入。任务入口、符号表和依赖边允许多处指向同一实体。采用 `$codemap` 时沿用其本层 Files/Subdirectories、父子路由及索引产物排除规则；其他地图保留等价归属，使用已有格式。
    +
    +以当前工作树双向核对：源码到地图查缺漏和重复归属，地图到源码查幽灵路径和失真的职责、接口、依赖及行范围。明确排除范围，不能通过扩大忽略规则掩盖缺漏。完成时简要说明增删改的处理或不适用依据，以及仍未闭合的缺口。
    +
    +双射只约束声明的索引范围。根地图统一记录排除项、仅保留边界的目录、原因及纳入例外；内部未索引的目录只保留唯一边界条目。采用 `$codemap` 时沿用其 Scope and Exclusions 格式，其他位置引用根地图的范围。
    +
    +忽略索引不等于过时、可删除或免于正确性验证。后续任务触及忽略区或依赖其契约时，读取所需代码并重新评估范围；决定变化时同步标记，不能把未索引内容宣称为已核对。维护脚本的扫描排除与地图的索引排除分别服务于变更检测和导航，不能未经判断直接互相套用。
    +
     使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    
     - 文件增删、移动、重命名：同步本层文件表、父目录路由及任务入口，移除已不存在目录的地图。
    @@ -43,6 +65,8 @@ python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    
     ## 实现与测试一起退役
    
    +**时效淘汰性**：每次变更主动复查受影响旧实现和临时层的保留依据、迁移状态与移除条件。替代实现已验证、消费者已迁移或旧需求已撤销时，在本次任务内完成退役；不能仅在地图上标成 deprecated 后无限保留。暂时仍需保留的分支必须记录具体消费者或当前契约及可检查的移除条件，在后续相关改动时重新核对。时效由当前需求和迁移证据判定，不按文件年龄或任意期限机械删除。
    +
     对候选项先核对用途、调用者、构建目标、平台和公开契约，再在本次授权范围内处理：
    
     - 已替换并完成迁移的实现：清除旧代码、专属包装层、构建入口、依赖、文档、测试与 fixture；历史由 Git 保留，不复制 `old/`、`v1/`、`backup/` 源码树。
    diff --git a/project-maintenance/references/verification.md b/project-maintenance/references/verification.md
    index 8966bb7..4abb151 100644
    --- a/project-maintenance/references/verification.md
    +++ b/project-maintenance/references/verification.md
    @@ -34,6 +34,12 @@ python3 <skill-dir>/scripts/maintenance_state.py record --root . --reviewed
     python3 <skill-dir>/scripts/maintenance_state.py check --root .
     ```
    
    +agent 首次读取可用 `inspect --root . --json --max-paths 20`；只取计数用 `--max-paths 0`。限量 JSON 的 `change_counts` 和 `separate_checkout_count` 始终覆盖完整扫描，`changes` 与 `separate_checkouts` 只包含本页。分页顺序为地图的增/改/删、其他文件的增/改/删、独立 checkout，各组按路径排序。
    +
    +按 `pagination.next_offset` 继续传入 `--offset`，并核对各页 `report_id` 一致；内容、审查记录或范围变化时重新取页。`pagination.total/returned/omitted/truncated` 明示完整量与本页量；计数模式不提供下一页，应改用正数限量。分页限制路径条目数量，路径长度和范围元数据仍会影响输出大小。省略 `--max-paths` 恢复原有完整 JSON；文本输出仍使用原有摘要。
    +
    +输出限量不改变扫描范围、审查记录或 `check` 退出码。不要把本页空列表、最后一页或截断结果当成无变更；以完整状态、计数和实际审查为准。`--root` 会解析为 Git 顶层，单个包的任务应再按地图与任务路径筛选，不能把仓库级记录说成仅审查了当前包。
    +
     - 扫描 Git 跟踪文件及未被忽略的新增文件，比较实际内容哈希；检测修改、增删、重命名及地图本身的变化。模式或时间戳相同的内容修改仍会触发。
     - tracked fixture、lockfile、配置与文档也在范围内。`CODEMAP.md`、`codemap.md` 和 `*.analysis.md` 在报告中单列；记录文件自身不进入指纹。
     - 默认排除缓存/依赖目录，以及根目录的 `build/`、`dist/`、`out/`、`coverage/`。若项目把真实源码放在这些位置，通过 `--include-generated` 纳入；该选择会记录在范围中。
    diff --git a/project-maintenance/scripts/maintenance_state.py b/project-maintenance/scripts/maintenance_state.py
    index 8098446..a7b01fb 100644
    --- a/project-maintenance/scripts/maintenance_state.py
    +++ b/project-maintenance/scripts/maintenance_state.py
    @@ -156,6 +156,46 @@ def write_state(root: Path, state: dict) -> None:
                 temporary.unlink()
    
    
    +def paginate_report(report: dict, max_paths: int, offset: int) -> dict:
    +    # Keep full counts and review status; pagination is presentation, not scope.
    +    entries = [
    +        (group, kind, name)
    +        for group in ("maps", "files")
    +        for kind, names in report["changes"][group].items()
    +        for name in names
    +    ]
    +    entries.extend(("separate_checkouts", "", item)
    +                   for item in report["separate_checkouts"])
    +    page = entries[offset:offset + max_paths]
    +    limited = {
    +        **report,
    +        "change_counts": {
    +            group: {kind: len(names) for kind, names in changes.items()}
    +            for group, changes in report["changes"].items()
    +        },
    +        "separate_checkout_count": len(report["separate_checkouts"]),
    +        "changes": {
    +            group: {kind: [] for kind in changes}
    +            for group, changes in report["changes"].items()
    +        },
    +        "separate_checkouts": [],
    +        "pagination": {
    +            "offset": offset, "limit": max_paths, "total": len(entries),
    +            "returned": len(page), "omitted": len(entries) - len(page),
    +            "truncated": len(page) < len(entries),
    +            "next_offset": offset + len(page) if page and (
    +                offset + len(page) < len(entries)
    +            ) else None,
    +        },
    +    }
    +    for group, kind, item in page:
    +        if group == "separate_checkouts":
    +            limited[group].append(item)
    +        else:
    +            limited["changes"][group][kind].append(item)
    +    return limited
    +
    +
     def print_report(report: dict, as_json: bool) -> None:
         if as_json:
             print(json.dumps(report, indent=2, sort_keys=True))
    @@ -183,7 +223,9 @@ def main() -> int:
         parser = argparse.ArgumentParser(description=__doc__)
         parser.add_argument("action", choices=("inspect", "record", "check"))
         parser.add_argument("--root", required=True, help="Path inside the intended Git project")
    -    parser.add_argument("--json", action="store_true", help="Emit complete structured output")
    +    parser.add_argument("--json", action="store_true", help="Emit structured output; complete unless --max-paths is set")
    +    parser.add_argument("--max-paths", type=int, help="Limit total JSON change/checkout entries; 0 emits counts only")
    +    parser.add_argument("--offset", type=int, default=0, help="Skip entries when using --max-paths")
         parser.add_argument("--reviewed", action="store_true", help="Record only after actual review")
         parser.add_argument("--exclude", action="append", help="Replace custom root-relative exclusions")
         parser.add_argument("--clear-excludes", action="store_true")
    @@ -196,6 +238,10 @@ def main() -> int:
             parser.error("--reviewed applies only to record")
         if args.exclude is not None and args.clear_excludes:
             parser.error("Use --exclude or --clear-excludes, not both")
    +    if args.max_paths is not None and (not args.json or args.max_paths < 0):
    +        parser.error("--max-paths requires --json and a nonnegative limit")
    +    if args.offset < 0 or (args.offset and args.max_paths is None):
    +        parser.error("--offset requires --max-paths and a nonnegative offset")
         try:
             root = repo_root(args.root)
             old = load_state(root)
    @@ -226,12 +272,22 @@ def main() -> int:
                 }
                 write_state(root, state)
                 status = "recorded-reviewed-state"
    -        print_report({
    +        report = {
                 "root": str(root), "status": status, "scope_changed": scope_changed,
                 "counts": {group: len(current[group]) for group in ("files", "maps")},
                 "changes": changes, "separate_checkouts": current["separate_checkouts"],
                 "excludes": patterns, "include_generated": include_generated,
    -        }, args.json)
    +        }
    +        if args.max_paths is not None:
    +            snapshot = {
    +                "current": current, "reviewed": old,
    +                "excludes": patterns, "include_generated": include_generated,
    +            }
    +            report["report_id"] = hashlib.sha256(
    +                json.dumps(snapshot, sort_keys=True).encode()
    +            ).hexdigest()
    +            report = paginate_report(report, args.max_paths, args.offset)
    +        print_report(report, args.json)
             return int(needs_review) if args.action == "check" else 0
         except (OSError, ValueError, UnicodeError) as error:
             if args.json:
     M codemap/SKILL.md
     M project-maintenance/SKILL.md
     M project-maintenance/references/verification.md
     M project-maintenance/scripts/maintenance_state.py
    ?? project-maintenance/references/automation.md
    wc: references/automation.md: No such file or directory
    nl: SKILL.md: No such file or directory
    nl: scripts/maintenance_state.py: No such file or directory
    ✓ • 16ms

## Activity

    file changes: Completed · 1 changes
    Update { move_path: None }: /home/fuurin/code/skills/project-maintenance/SKILL.md
    @@ -26,3 +26,3 @@
     
    -用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
    +用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。已有格式的日常维护只读取命中的地图部分；首次建图或需要格式规则时再加载 `$codemap`。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
     

## Activity

    $ python3 /home/fuurin/.codex/skills/.system/skill-creator/scripts/quick_validate.py .; git diff --check; nl -ba SKILL.md | sed -n '16,36p'; nl -ba references/automation.md | sed -n '1,60p'; python3 - <<'PY'
        from pathlib import Path
        import re
        files = [Path('SKILL.md'), Path('references/verification.md'), Path('references/automation.md')]
        checked = 0
        for path in files:
            for target in re.findall(r'\[[^\]]*\]\(([^)]+)\)', path.read_text()):
                if '://' not in target and not target.startswith('#'):
                    assert (path.parent / target.split('#', 1)[0]).exists(), (path, target)
                    checked += 1
        print(f'Local Markdown links: {checked} valid')
        PY
    Skill is valid!
        16
        17    Git 项目可先运行只读检查：
        18
        19    ```bash
        20    python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root> --json --max-paths 20
        21    ```
        22
        23    脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。限量输出仍保留完整计数和审查状态；分页或先只取计数的方式见[验证参考](references/verification.md)。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
        24
        25    ## 工具与 agent 协作
        26
        27    用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。已有格式的日常维护只读取命中的地图部分；首次建图或需要格式规则时再加载 `$codemap`。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
        28
        29    先取计数、路径和行范围，再读必要内容；完整扫描结果可以留在工具侧。按任务需要选择 `rg`、AST、语言服务器或动态脚本，复用工具及维护状态入口。需要选型、批量核对或自建脚本时，读取[自动化与 codemap 协作](references/automation.md)。
        30
        31    ## 随改动更新地图
        32
        33    ### 双射性与增删改
        34
        35    每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
        36
         1    # 自动化与 codemap 协作
         2
         3    需要选择工具、批量核对或动态生成辅助脚本时读本参考。工具提取可重复的事实，agent 判断当前契约与变更影响；读取范围随证据扩展。本文描述两种技能的协作，地图格式与导航规则由可用的 `$codemap` 或项目现有地图维护。
         4
         5    ## 已有能力及归属
         6
         7    | 能力 | 复用入口 | project-maintenance 的工作 |
         8    | --- | --- | --- |
         9    | 任务路由、Domain 过滤、条件读取消费者 | codemap 的 Task Guide、Files、Dependencies | 用变更证据选择入口，判断公开契约是否受影响 |
        10    | 大文件功能到行范围的定位 | codemap 的 `.analysis.md` Feature Index | 核对受影响范围仍对应当前源码 |
        11    | 文件/目录唯一归属、范围和排除项 | codemap 的 Files、Subdirectories、根 Scope and Exclusions | 检查当前工作树的缺漏、重复、幽灵路径并同步 |
        12    | 地图增删改与父子路由更新 | codemap 维护模式 | 将地图更新纳入实现、文档、构建和验证的完成条件 |
        13    | 审查之后发生了哪些变化 | `maintenance_state.py` 的 inspect/check | 结合 Git 差异区分本次任务和其他改动 |
        14    | 旧实现、测试和迁移层是否应退役 | 当前源码、消费者、产品契约及验证入口 | agent 判断保留依据与移除条件，执行获授权的维护 |
        15
        16    维护状态的扫描范围与地图的索引范围分别保存在审查记录和根地图中；扫描会包含许多无需索引的配置、fixture 和文档。临时候选清单用于本次操作，不成为另一份权威地图。采用 codemap 时使用其维护模式；普通维护沿用既有配置，只有新建或改变索引范围时才处理相应配置问题。
        17
        18    ## 按需求选择工具
        19
        20    | 要解决的问题 | 工具与使用条件 | 返回给 agent 的信息 |
        21    | --- | --- | --- |
        22    | 找文件及本次差异 | Git、`rg --files`；审查基线用已有状态脚本 | 状态、数量、路径、差异片段 |
        23    | 找名称、路径和文本消费者 | `rg -l` 先收候选，`rg -n -F` 再定位；按目录/类型缩小范围 | 去重路径及必要的匹配行 |
        24    | 找语法结构、批量迁移同类调用 | 有需要时使用 ast-grep 的结构搜索、outline、重写预览 | 符号/节点、文件、范围、替换差异 |
        25    | 分辨同名符号、查询引用、语义重命名 | 项目已有的语言服务器，例如 clangd、rust-analyzer；配置、feature 和索引覆盖需有效 | 真实符号的定义与引用位置 |
        26    | 跨语言声明定位，缺少现成结构提取器 | 已安装的 Universal Ctags；Python 可直接用标准库 AST | 符号及行号，作为导航候选 |
        27    | 地图表格与当前目录的批量集合核对 | 先复用 codemap/项目工具；缺少时生成限定格式的临时比较脚本 | 缺漏、重复归属、幽灵路径、越界行范围 |
        28    | 重复全量提取确实耗时 | 优先复用语言服务器缓存；有测量依据再考虑 SQLite 存放按内容哈希更新的事实 | 与本次变更相关的行，标明来源和有效版本 |
        29
        30    ast-grep 支持结构搜索、JSON 输出和重写；当前官方版本还提供 outline。使用前检查本地 `--help` 的能力，缺少工具时先选已安装且足够的方式。[官方 CLI](https://ast-grep.github.io/reference/cli.html) 与[工具指南](https://ast-grep.github.io/guide/tooling-overview.html)。Universal Ctags 的 JSON 支持取决于编译选项，其符号清单用于定位。[官方 JSON 文档](https://docs.ctags.io/en/latest/man/ctags-json-output.5.html)。clangd 的后台索引提供项目引用信息，当前打开文件的动态索引用于跟随编辑变化。[索引设计](https://clangd.llvm.org/design/indexing)。
        31
        32    结构匹配和文本命中都只是证据。无命中时核对搜索忽略项、动态注册、FFI、导出接口和有效平台分支；单凭它们不执行删除。语言服务器也只覆盖其有效配置下的源码。源码内容变化后，旧行号、片段和提取结果需要重新核对。
        33
        34    ## 一次维护怎样执行
        35
        36    1. **取增量事实**：用 `inspect --json --max-paths 20` 取得完整计数和首批路径，必要时分页。首次没有审查记录时，清单表示待建立基线，不能把全部 added 视为本次新增；结合 Git 的暂存、未暂存差异与未跟踪文件判断。
        37    2. **通过地图定向**：沿 codemap 的任务路由进入受影响目录，用 Feature Index 确定大文件行范围。有具体契约缺口再查消费者，消费者还可能位于本次未修改的目录。
        38    3. **取得最小证据**：批量运行独立查询，工具侧去重和筛选；返回路径、符号、行号和所需片段。完整清单较大时保存在临时文件并明示截断，读取后续项或按任务筛选，不能用一个 `head` 隐藏未知范围。
        39    4. **agent 判断并执行增删改**：以实现、消费者和当前契约决定操作。适用的统一替换先看匹配与差异预览，再执行受限的批量编辑；任意字符串替换无法代替语义重命名。
        40    5. **闭合影响**：同步 codemap、相关说明、构建路线和有效验证。完成实际审查后才 record，再 check；分页参数只压缩报告，不缩小记录范围。
        41
        42    例如先用 `rg -l -F -e 'OldEngine' src tests` 取得候选路径；再对选定文件定位行段。具体命中需要 agent 判断，不能把测试字符串、注释和实际调用混为一个消费者。Git 与 rg 的文件过滤结果也不能直接充当地图的声明范围。
        43
        44    ## 动态自建工具的条件
        45
        46    当既有命令的组合已经够用时直接组合。有明确的重复计算或格式转换需求时，再写一个窄小辅助脚本。例如 codemap 格式的 Files/Subdirectories 双向集合比较应复用或补到 codemap 的工具归属；维护状态继续使用本技能的单一脚本入口。
        47
        48    临时脚本放在获允许的临时目录或项目既有本地工具目录。参数明确给出根目录、输入、声明的范围、格式和输出位置；未知表格格式要报“未解析”，不能假装核对成功。只比较权威清单条目，避免把 Task Guide、Key Exports 和导航链接算作重复归属。范围、例外和仅保留边界需依据根地图解释，不能照搬维护扫描的排除项。
        49
        50    把事实与候选判断分开输出，保留来源路径、当前内容标识和必要行号；输出包括完整计数、返回数量、遗漏/截断、后续读取入口和错误。解析、读取或子进程失败时返回失败；参数化调用工具，避免将仓库文本拼成可执行的 shell 命令。批量写入使用具体编辑清单或补丁，并核对预期内容仍匹配当前文件。
        51
        52    只用于一次任务的脚本无需长期维护。经实际复用证明有收益的脚本再归入对应 skill/项目工具，并补充使用说明和行为验证；保留一套接口。节省 token 主要靠减少返回的无关内容和重复读取；多 agent 会增加总调用量，只有已获授权且任务能独立分工时，才用它隔离中间上下文。
    Local Markdown links: 3 valid
    ✓ • 0ms

## Assistant

已对照，协作方式确定为：

| 职责 | 复用方式 |
|---|---|
| 定位代码、维护地图与索引范围 | codemap |
| 检测变更、记录审查状态 | 现有维护脚本 |
| 判断影响、执行增删改及退役 | agent |

工具优先用 **Git + rg**；结构搜索和批量改写按需使用 [ast-grep](https://ast-grep.github.io/reference/cli.html)。动态脚本只补具体的重复计算或格式转换，复用有效后再纳入对应工具入口。

已写入[协作与工具方案](/home/fuurin/code/skills/project-maintenance/references/automation.md)，并给[维护脚本](/home/fuurin/code/skills/project-maintenance/scripts/maintenance_state.py:159)增加分页。当前仓库首批 20 条的输出从 **4340 降至 1857 字符，减少约 57%**，完整计数和审查状态保留；分页及相关行为验证通过。

另外，codemap README 有部分旧规则，本次对照以 SKILL.md 为准。

## User

ast-grep好用吗

## Assistant

我会结合官方文档和刚才的维护场景，判断它在哪些操作上值得用，以及和 rg、语言服务器怎么分工。

## Activity

    Opened https://ast-grep.github.io/advanced/comparison.html

## Activity

    Searched the web

## Activity

    $ command -v ast-grep || true; command -v sg || true; npx --yes ctx7@latest docs /ast-grep/ast-grep.github.io 'Strengths and limitations of structural search versus ripgrep and language server semantic rename; Python function call pattern metavariables'
    npm error code EPERM
    npm error syscall connect
    npm error errno EPERM
    npm error FetchError: request to https://registry.npmjs.org/ctx7 failed, reason: connect EPERM 127.0.0.1:7890 - Local (undefined:undefined)
    npm error     at ClientRequest.<anonymous> (/home/fuurin/.nvm/versions/node/v24.20.0/lib/node_modules/npm/node_modules/minipass-fetch/lib/index.js:130:14)
    npm error     at ClientRequest.emit (node:events:514:28)
    npm error     at emitErrorEvent (node:_http_client:114:11)
    npm error     at _destroy (node:_http_client:1167:9)
    npm error     at onSocketNT (node:_http_client:1188:5)
    npm error     at process.processTicksAndRejections (node:internal/process/task_queues:91:21) {
    npm error   code: 'EPERM',
    npm error   errno: 'EPERM',
    npm error   syscall: 'connect',
    npm error   address: '127.0.0.1',
    npm error   port: 7890,
    npm error   type: 'system'
    npm error }
    npm error
    npm error The operation was rejected by your operating system.
    npm error It is likely you do not have the permissions to access this file as the current user
    npm error
    npm error If you believe this might be a permissions issue, please double-check the permissions of the file and its containing directories, or try running the command again as root/Administrator.
    npm error Log files were not written due to an error writing to the directory: /home/fuurin/.npm/_logs
    npm error You can rerun the command with `--loglevel=verbose` to see the logs in your terminal
    ✗ (1) • 1m 10s

## Activity

    $ npx --yes ctx7@latest docs /ast-grep/ast-grep.github.io 'Strengths and limitations of structural search versus ripgrep and language server semantic rename; Python function call pattern metavariables'
    ### Semgrep Pattern Examples: Ellipsis Metavariables, Typed Metavariables, Deep Expression Operators
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/blog/code-search-design-space.md
    
    Semgrep pattern examples demonstrating ellipsis metavariables, typed metavariables, and deep expression operators. These patterns extend standard programming language syntax and are not valid code in the target language.
    
    ```yaml
    # combine ellipses and metavariables to match a sequence of ASTs
    # note the ellipsis is not valid programming language syntax
    pattern: foo($...ARGS, 3, $...ARGS)
    # this pattern will match foo(1, 2, 3, 4, 5)
    ```
    
    ```yaml
    #  look for calls to the log method on Logger objects.
    # A simple pattern like this will match `Math.log()` as well
    pattern: $LOGGER.log(...)
    # typed metavariable can put a type constraint on the metavariable
    # but it is no longer valid Java code
    pattern: (java.util.logging.Logger $LOGGER).log(...)
    ```
    
    ```yaml
    # Use the deep expression operator <... [your_pattern] ...>
    # to match an expression that
    # could be deeply nested within another expression
    pattern: |
      if <... $USER.is_admin() ...>:
        ...
    ```
    
    --------------------------------
    
    ### Search with Rule using keyword arguments
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/guide/api-usage/py-api.md
    
    Demonstrates using keyword arguments to `find` and `find_all` to match nodes by pattern and kind. Multiple keyword arguments are combined with AND logic. The example shows that a pattern like `print($A)` matches the call, `kind="string"` matches the string literal, and combining them with `kind="string"` returns `None` because the pattern is not a string literal.
    
    ```python
    root = SgRoot("print('hello world')", "python")
    node = root.root()
    node.find(pattern="print($A)") # will return the print function call
    node.find(kind="string") # will return the string 'hello world'
    # below will return print function call because it matches both rules
    node.find(pattern="print($A)", kind="call")
    # below will return None because the pattern cannot be a string literal
    node.find(pattern="print($A)", kind="string")
    
    strings = node.find_all(kind="string") # will return [SgNode("hello world")]
    assert len(strings) == 1
    ```
    
    ### Comby
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/advanced/tool-comparison.md
    
    Comby is a fast and flexible tool that uses structural patterns to match and rewrite code across languages and file formats.
    
    **Pros**:
    * It does not rely on language-specific parsers, which makes it more generic and robust. It can handle any language and file format, including non-code files like JSON or Markdown.
    * It has a custom syntax for specifying patterns and replacements, which can handle various syntactic variations and transformations.
    
    
    **Cons**:
    * It is not aware of the syntax and semantics of the target language, which limits its expressiveness and accuracy. It may miss some matches or generate invalid code due to syntactic or semantic differences.
    * It does not support indentation-sensitive languages like Python or Haskell, which require special handling for whitespace and indentation.
    * It is hard to write complex queries with Comby, such as finding a function that does not call another function. It does not support logical operators or filters for patterns.
    
    --------------------------------
    
    ### Textual vs Structural
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/advanced/core-concepts.md
    
    ## Textual vs Structural
    
    When you use ast-grep to search for patterns in source code, you need to understand the difference between textual and structural matching.
    
    Source code input is text, a sequence of characters that follows certain syntax rules. You can use common search tools like [silver-searcher](https://github.com/ggreer/the_silver_searcher) or [ripgrep](https://github.com/BurntSushi/ripgrep) to search for text patterns in source code.
    
    However, ast-grep does not match patterns against the text directly. Instead, it parses the text into a tree structure that represents the syntax of the code. This allows ast-grep to match patterns based on the structure of the code, not just its surface appearance. This is known as [structural](https://docs.sourcegraph.com/code_search/reference/structural) [search](https://docs.sourcegraph.com/code_search/reference/structural), which searches for code with a specific structure, not just a specific text.
    
    _Therefore, the patterns you write must also be of valid syntax that can be compared with the code tree._
    
    :::tip Textual Search in ast-grep
    Though `pattern` structurally matches code, you can use [the atomic rule `regex`](/guide/rule-config/atomic-rule#regex) to search the node's full text, including its children. This way, it is possible to combine textual and structural matching in ast-grep.
    :::
    
    --------------------------------
    
    ### Formal Queries Based on Existing Programming Languages > Code with Holes
    
    Source: https://github.com/ast-grep/ast-grep.github.io/blob/main/website/blog/code-search-design-space.md
    
    ### Code with Holes
    
    This approach involves providing code snippets with placeholders to search for code fragments.  For example, a user might search for the following pattern in Java:
    
    ```java
    public void actionClose (JButton a, JFrame f) {
     $$$BODY
    }
    ```
    
    Here, `$$$BODY` is a placeholder, and the code search engine will try to locate all matching code. ast-grep falls into this category, treating the query as an Abstract Syntax Tree (AST) with holes. The holes in ast-grep are called metavariables.
    
    Other tools like gritql and the [structural search feature](https://www.jetbrains.com/help/idea/tutorial-work-with-structural-search-and-replace.html) in IntelliJ IDEA also use this technique.
    ✓ • 5.02s

## Assistant

文档确认，它能用代码形状匹配跨行调用，并避开注释和字符串里的同名文本。需要注意的边界是：它主要分析语法结构，同名函数的真实归属、类型和完整引用关系仍要结合语言服务器或 agent 判断。

## Assistant

**好用，尤其适合我们这里的结构化搜索和批量 API 迁移。**

它可以用 `OldEngine($$$ARGS)` 这样的代码模式找到调用，处理跨行、不同空白，并避开注释和字符串里的同名文本。简单查询比较直观。[模式文档](https://ast-grep.github.io/guide/pattern-syntax.html)

它还支持替换预览和 JSON 输出，方便 agent 先检查匹配，再批量修改；返回给 agent 时只保留路径、行号和必要片段，有助于节省 token。[CLI 文档](https://ast-grep.github.io/reference/cli.html)

主要限制是**语义分析有限**：类型、真实符号归属、完整调用关系需要其他证据；因此无匹配不能直接作为删除依据。[官方说明](https://ast-grep.github.io/advanced/tool-comparison)

我的建议是：**codemap 定位范围，rg 查文本，ast-grep 查结构与统一改写，语言服务器查符号关系，agent 决定如何维护。**

## User

好的，同步code map和project maintain，裁剪掉不必要的部分

## Assistant

我会同步两份技能，精简重复的导航、索引和维护说明，同时保留增删改、退役依据及验证要求。这次会用 skill-creator 整理技能入口，并按 codemap 的现有规则核对两边的职责。

## Activity

    $ for path in ../AGENTS.md ../codemap/AGENTS.md ../codemap/references/AGENTS.md AGENTS.md references/AGENTS.md scripts/AGENTS.md; do if [ -f "$path" ]; then printf '\n%s\n' "$path"; cat "$path"; fi; done; readlink -f /home/fuurin/.codex/skills/codemap; readlink -f /home/fuurin/.codex/skills/project-maintenance; sed -n '1,145p' /home/fuurin/.codex/skills/.system/skill-creator/SKILL.md
    zsh:1: command not found: readlink
    zsh:1: command not found: readlink
    zsh:1: command not found: sed
    ✗ (127) • 0ms

## Activity

    $ cat ../codemap/SKILL.md; sed -n '1,190p' ../codemap/README_EN.md; git diff -- ../codemap/SKILL.md SKILL.md
    ---
    name: codemap
    description: Use when indexing a codebase for agent navigation, generating or refreshing CODEMAP.md files, mapping large project structure, or adding CODEMAP guidance for learning/maintenance workflows.
    ---
    
    # CODEMAP — Codebase Navigation Index Generator
    
    Generate hierarchical `CODEMAP.md` files that help agents locate relevant code without scanning unrelated files. Core navigation: **Task Guide first → Domain filter → targeted reads**. For files over 1000 lines, generate a companion `<filename>.analysis.md` with intent-to-line-range mapping.
    
    ## Core Principles
    
    - `CODEMAP.md` is a navigation constraint, not documentation to browse.
    - Prefer positive guidance (Task Guide, Domain, Key Exports) over broad listings.
    - Dependencies are a safety net, not an invitation to chain-read.
    - Each CODEMAP describes only its own directory level. Child directory details belong in child CODEMAPs.
    - Maintain a bijection between in-scope source entities and canonical inventory entries: no omissions, no duplicate ownership, no entries for nonexistent entities. Navigation references may be many-to-one; they are not additional inventory entries.
    - Every code change requires considering addition, deletion, and modification together. Maintenance also actively retires superseded code within the authorized task scope; adding an index entry does not complete a migration.
    
    ## Bijection and Freshness
    
    Define the inventory scope using the declared ignore rules. Every included immediate source file has exactly one canonical row in its directory's Files table; every included immediate child directory has exactly one Subdirectories row. Parent maps route to children without duplicating their inventories. CODEMAPs and companion analysis files are index artifacts, not source entities to recursively index. Key Exports and Task Guide are selective navigation views, not exhaustive inventories of all symbols or tasks.
    
    Validate both directions against the current source tree, including uncommitted changes: source → map catches omissions and duplicate ownership; map → source catches stale paths, symbols, responsibilities, dependencies, and line ranges. Do not exclude a source merely to hide an omission. Shared implementations can have multiple callers and navigation references without duplicate canonical entries.
    
    Freshness is a completion condition after each code change, not a timestamp update. In maintenance mode, inspect affected implementations for replacement, duplication, obsolete requirements, and completed migration scaffolding. Retire confirmed obsolete code and its exclusive tests, dependencies, build routes, documentation, and map entries in the same task. Retained compatibility paths need concrete consumers or a current contract and an explicit removal condition; recheck that condition when affected. Age or absence of local text references alone is not proof of obsolescence. Learning mode remains read-only for source code.
    
    ## Language Rule
    
    Generated files use the user's request language for prose/headings. Code identifiers, file names, paths, and symbols keep original spelling.
    
    ## Before Generation: Ask Three Questions
    
    Unless already specified:
    
    1. **Mode**: `Learning` (read-only study) or `Maintenance` (active development).
    2. **Sub-agents**: `Yes, max 3` (recommended), custom limit, or `No`.
    3. **Ignore rules**: `Defaults + .gitignore` (recommended), or add custom patterns.
    
    Mode differences:
    
    | Aspect | Learning | Maintenance |
    |---|---|---|
    | Frontmatter | `mode: learning` | `mode: maintenance`, `commit: <hash>` |
    | Task Guide | suggested entry point | primary navigation, strict |
    | Domain | soft focus hint | hard filter unless justified |
    | Dependencies | reference material | gated by interface/impact rules |
    | Updates | one-time | incremental after code changes |
    
    ## Ignore Rules
    
    Merge in order:
    1. Built-ins: `.git/`, dependency dirs, virtualenvs, build outputs, caches, logs, lockfiles, minified files, binaries, image/font assets, IDE folders.
    2. Project `.gitignore`.
    3. User custom patterns.
    
    Include generated code only if it affects navigation; mark `Generated, do not edit manually`.
    
    Bijection applies only within the declared inventory scope. Low-priority areas may be deliberately excluded or represented only by a directory boundary, according to the user's priorities and project conventions. Record these choices visibly in the root CODEMAP's **Scope and Exclusions** section; never silently omit them.
    
    Use `Path / pattern | Treatment | Reason / basis` rows. Treatment is either `Excluded` (no inventory obligation inside this scope) or `Boundary only` (retain one directory entry marked `Internals not indexed`, without a child map or internal file inventory). Patterns are root-relative; list any included exceptions explicitly. Record default exclusions and `.gitignore` as identifiable rule sources, grouping routine patterns rather than listing every ignored file. Project-specific omissions need concrete paths/patterns and reasons, not merely “defaults”. This section is authoritative; frontmatter `ignore` summarizes or points to it.
    
    Ignoring indexing does not mean code is obsolete, safe to delete, or exempt from correctness checks. If a task touches an excluded area or depends on its contract, inspect what is needed and reconsider the recorded scope. Update the scope decision if it changes; do not claim unindexed internals were verified.
    
    ## Generation Workflow
    
    ### 1. Build Global Context
    
    Read lightweight project context only:
    - Prefer root `README.md` / `README.rst` / `README.txt`.
    - Else metadata: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, etc.
    - Else infer from structure; mark guesses `inferred, verify against code`.
    
    Produce: purpose, architecture shape, major domains. Omit badges/changelogs.
    
    ### 2. Scan and Measure
    
    After applying ignore rules:
    - Build filtered directory topology.
    - Count source files, lines, size per first-level subdirectory and project total.
    - Files over 1000 lines:
      - `<=5` → generate all `.analysis.md`.
      - `>5` → ask: all, top 5, selected, or none.
    
    ### 3. Dispatch Work
    
    If sub-agents enabled, choose count `K`:
    
    | Project size | K |
    |---|---|
    | `<=3000` lines or `<=500KB` | 1 |
    | `3001-15000` lines or `500KB-3MB` | `min(N, 2)` |
    | `>15000` lines or `>3MB` | `N` |
    
    If line count and size disagree, use the larger `K`. Assign first-level directories by greedy bin packing. Keep root loose files with main agent if small (`<=200` lines), else assign to lightest bin.
    
    Sub-agent prompts: self-contained, plain English. Include compressed global context, output language, ignore rules, assigned directories, large files list, required formats, forbidden paths. No overlapping write ownership.
    
    ### 4. Generate Per-Directory Maps
    
    One `CODEMAP.md` per fully indexed source directory (root + included subdirectories). Excluded and boundary-only interiors do not require maps. Each map describes only the current directory level.
    
    ### 5. Assemble Root and Install Protocol
    
    Read first-level CODEMAP summaries and Task Guides. Write root `CODEMAP.md`, then install the Navigation Protocol block into `AGENTS.md` (preferred) or `CLAUDE.md`. Replace existing block if present; do not append duplicates. If neither file exists, create `AGENTS.md`.
    
    ---
    
    ## CODEMAP.md Structure
    
    ### Frontmatter
    
    ```yaml
    ---
    mode: learning | maintenance
    commit: abc1234f        # maintenance only
    ignore: ...             # root only
    generated_at: YYYY-MM-DD
    stats:                  # root only
      total_files: 114
      total_lines: 18200
      total_size: 4.2 MB
    ---
    ```
    
    ### Sections
    
    Sections appear in this fixed order. Omit a section when it would be empty.
    
    | Section | Root | Mid-level | Leaf | Container |
    |---|---|---|---|---|
    | Summary | 1 sentence | 1 sentence | 1 sentence | 1 sentence |
    | Scope and Exclusions | yes | — | — | root only |
    | Task Guide | yes | yes | yes | yes |
    | Subdirectories | yes | yes | — | yes |
    | Key Exports | yes | yes | if needed | — |
    | Files | yes | yes | yes | — |
    | File Dependencies | — | immediate files only | yes | — |
    
    **Container directory** = directory with no source files, only subdirectories. Generate only Summary + Task Guide + Subdirectories.
    Root containers also retain Scope and Exclusions, so deliberate omissions remain visible.
    
    ### Summary
    
    One sentence. Mark uncertain inference with `(inferred)`.
    
    ### Task Guide
    
    Columns: `Task | Domain | Target | Also Check`.
    
    Rules:
    - Each row is a concrete scenario, not a vague category.
    - Learning mode: understanding intents (`理解认证流程`). Maintenance mode: modification intents (`新增认证方式`).
    - `Domain` must match values used in the same map's Files or Subdirectories tables.
    - `Target` = primary read set. `Also Check` = conditional candidate, not automatic read list. Keep short and empirical.
    - Ensure every functional subdirectory and every common modification scenario has at least one row.
    
    Task Guide interpretation by mode:
    
    - **Maintenance**: Read `Target` first. Read `Also Check` only when the task explicitly mentions it, target code proves it needed, or public contract impact requires it. If no row matches, filter by `Domain`; non-matching domains excluded unless justified.
    - **Learning**: Use `Target` as starting point. Treat `Also Check` as optional reference.
    
    ### Subdirectories
    
    Columns: `Dir | Domain | Depends On | Purpose`.
    
    Rules:
    - `Domain`: short functional area (e.g., `Auth`, `Prompt`, `Tool System`, `MCP`, `Runtime`).
    - **Domain granularity**: each functionally distinct subdirectory MUST have a unique Domain value. Never assign a single generic Domain (e.g., `LLM Integration`) to all entries.
    - `Depends On`: directory-level dependencies (internal and external). Use `—` if none.
    - `Purpose`: one sentence.
    
    ### Key Exports
    
    Columns: `Symbol | Source | Line`.
    
    Rules:
    - Only symbols used by **other directories**. Internal-only symbols belong in child CODEMAPs.
    - Line numbers as `L:<number>`. Sort by architectural importance.
    - Cap at ~15 entries. Child CODEMAPs handle detailed symbols.
    
    ### Files
    
    Columns vary by directory type:
    
    - **Root**: `File | Domain | Function`
    - **Mid-level / Leaf**: `File | Domain | Deps | Function`
    
    `Deps` column (compact notation):
    - `←` = files outside this directory that this file imports.
    - `→` = files outside this directory that depend on this file.
    - Example: `← core/errors.py | → main.py, chat/service.py`
    - `→` >5 files: `→ N files (foundational); rg "SymbolName" --type py -l`
    - Omit `Deps` column entirely when no file in the directory has cross-dir dependencies. Use `—` for individual files with no deps.
    
    Rules:
    - One row per immediate file. **Never list files from child directories that have their own CODEMAP.**
    - `Domain` must match local Task Guide / Subdirectories values. `—` for trivial re-export files.
    - `Function`: one concise sentence. For large files, append `→ see <filename>.analysis.md`.
    - Include pure re-export files such as `__init__.py` once in Files; Key Exports does not replace their canonical inventory row.
    
    ### File Dependencies
    
    Columns: `File | Imports (in-dir) | Exposed To (in-dir)`.
    
    Rules:
    - Same-directory relationships only. Only for immediate files (not child directory files).
    - Mid-level directories: list only files directly in the directory itself, never files in child subdirectories.
    - Maintenance mode: read `Imports` only when an imported interface contract is unclear. Read `Exposed To` only when changing a public signature, return type, or documented semantics.
    - Learning mode: reference only; do not chain-read unless current logic is unclear without it.
    
    ### Parent-Child Decoupling
    
    When a child directory has its own CODEMAP:
    - Parent does NOT list the child's internal files in Files, File Dependencies, or Key Exports.
    - Parent only references the child directory in Subdirectories and Task Guide.
    - Child-internal symbols appear only in the child's Key Exports.
    
    ---
    
    ## Large File Analysis (.analysis.md)
    
    For source files over 1000 lines, create `<filename>.analysis.md` beside it.
    
    ### Structure
    
    ```markdown
    ---
    source: filename.py
    lines: 1842
    generated_at: YYYY-MM-DD
    ---
    
    > One-sentence summary.
    
    ## Feature Index
    
    | Intent | Lines | Notes |
    |---|---|---|
    
    ## Symbols
    
    | Symbol | Type | Line |
    |---|---|---|
    
    ## Logical Sections
    
    | Lines | Content |
    |---|---|
    ```
    
    ### Rules
    
    - **Feature Index** is primary. Map concrete learning or development intents to exact line ranges. Notes: same-file coupling only. If no intent maps to a section, omit that row.
    - **Symbols**: top-level public symbols only (classes, functions, constants). Not internal helpers.
    - **Logical Sections**: high-level structural segments only, **5-10 rows max**. Provides structural overview and serves as fallback when Feature Index has no match. Do not expand to function-level granularity.
    - Optional **Class Hierarchy** only when inheritance depth >2.
    - No code snippets, API signatures, or implementation detail paragraphs.
    - In maintenance mode, agents read only matched line ranges from Feature Index; Logical Sections is the fallback.
    
    ---
    
    ## Navigation Protocol (Project-Level Injection)
    
    Install exactly one protocol block into project `AGENTS.md` or `CLAUDE.md`. Choose by mode.
    
    ### Learning Mode Block
    
    ````markdown
    ## CODEMAP Navigation Protocol
    
    This project uses hierarchical `CODEMAP.md` index files for code navigation. Files over 1000 lines may have companion `.analysis.md` structural maps.
    
    ### Navigation Rules
    
    1. Start from root `CODEMAP.md`. Read Task Guide first.
    2. Task Guide match: Target = primary read set. Also Check = conditional candidates (decide after reading Target).
    3. No Task Guide match → filter Subdirectories by Domain, enter only matching-domain subdirectories.
    4. Drill down layer by layer; consult local Task Guide at each level before reading source files.
    5. Container directories (no source files): read only Task Guide + Subdirectories.
    6. Large files: read `.analysis.md` Feature Index first, match Intent to line ranges. Use Logical Sections as fallback.
    7. Batch-read final target files in parallel.
    8. No speculative expansion: extend read set only when already-read code proves the need.
    ````
    
    ### Maintenance Mode Block
    
    ````markdown
    ## CODEMAP Navigation Protocol
    
    This project uses hierarchical `CODEMAP.md` index files for code navigation. Files over 1000 lines may have companion `.analysis.md` structural maps. For development tasks, these rules are strict navigation constraints.
    
    ### Navigation Rules
    
    1. Start from root `CODEMAP.md`. Read Task Guide first.
    2. Task Guide match: Target = primary read set. Read Also Check only when the task explicitly involves it, target code proves the need, or public contract impact requires it.
    3. No Task Guide match → filter Subdirectories by Domain. Non-matching domains are excluded unless already-read code gives a concrete reason.
    4. Drill down layer by layer; consult local Task Guide at each level before reading source files.
    5. Container directories (no source files): read only Task Guide + Subdirectories.
    6. Large files: read `.analysis.md` Feature Index first, match Intent to line ranges. Use Logical Sections as fallback.
    7. Batch-read final target files in parallel.
    8. No speculative expansion: each additional file requires an explicit reason.
    
    ### Dependency Gating
    
    The Deps column in Files tables marks cross-directory dependencies:
    - `←` (imports): read only when the imported interface contract is needed to understand the current file.
    - `→` (exposed to) ≤5 files: read only when changing a public signature, return type, or documented semantics.
    - `→` >5 files (foundational): run the search command provided in CODEMAP, filter by Domain, then read only justified matches.
    - No chaining: do not read dependencies-of-dependencies unless a specific contract gap remains.
    
    ### Update Rules
    
    Every code change requires evaluating all three operations within the affected task scope; perform each applicable operation without waiting for a separate cleanup request:
    - Add: supply missing implementation or contract coverage and missing map entries for in-scope files/directories.
    - Delete: remove confirmed redundant or superseded implementations and their exclusive tests, dependencies, build routes, documentation, and stale map/analysis entries. Check real consumers, supported platforms, dynamic/FFI entry points, and public contracts first.
    - Modify: update existing implementations and canonical entries in place when responsibilities, interfaces, dependencies, paths, or analysis line ranges change. Moves/renames remove old routes and install current ones, including parent Task Guide/Subdirectories references.
    
    Completion requires a bijection between included source files/directories and their canonical local Files/Subdirectories entries: each entity appears exactly once, each entry resolves to a current entity, and excluded scope is explicit. Task Guide, Key Exports, and dependency links may reference the same entity; they must remain accurate without duplicating inventory ownership. Check source → map for missing/duplicate entries and map → source for stale entries and semantics, using the working tree rather than just the last commit.
    
    Deliberately unindexed areas must appear in root CODEMAP's Scope and Exclusions with root-relative paths/patterns, treatment (`Excluded` or `Boundary only`), and reason/basis. Boundary-only directories retain a marked directory entry, without an internal inventory obligation. Record rule sources for routine exclusions and explicit exceptions. When a task touches these areas or depends on their contracts, inspect the necessary code and reconsider scope; exclusion is neither deletion evidence nor a correctness exemption.
    
    Actively recheck retirement conditions for affected compatibility layers and migration code. Once replacement and consumer migration are verified, remove the old path in this task; retained exceptions need concrete consumers or a current contract and a removal condition. Do not accumulate `old/`, `v1/`, or `backup/` copies. Age or no local text references alone does not justify deletion.
    
    Pure internal equivalent changes may leave maps unchanged only after verifying that indexed responsibilities, contracts, dependencies, symbols, and line ranges remain accurate. Briefly report applicable additions/deletions/modifications, reasons for no action, and unresolved gaps; timestamps and passing builds alone do not establish map accuracy.
    ````
    
    ---
    
    ## Edge Cases
    
    - **Monorepo**: map each package root plus a top-level package index.
    - **Deep nesting**: layer-by-layer drill-down; each level's map stays local.
    - **Huge flat directory** (`>200` files): group Files rows by Domain subheadings.
    - **Generated code**: include only when navigation needs it; mark `Generated, do not edit manually`.
    - **No metadata/README**: infer cautiously; mark uncertainty.
    - **Task Guide gaps**: acceptable. Fall back to Domain filtering and Key Exports.
    - **Foundational files** (>5 dependents): provide grep command, require Domain filtering.
    # codemap-skill
    
    **[中文](README.md)** | **English**
    
    ---
    
    A **hierarchical codebase navigation index** skill for Claude Code and other AI coding agents.
    
    Generates `CODEMAP.md` index files in the project root and every source subdirectory, plus `<filename>.analysis.md` deep-analysis companion files for extra-large source files (>1000 lines). Uses a **constraint-based navigation strategy** — Task Guide → Domain filter → dependency safety net — to prevent agents from reading irrelevant files.
    
    ## Features
    
    ### Index & Navigation
    - **Hierarchical indexing**: Root + per-subdirectory `CODEMAP.md` with simplified directory structure and file/subdirectory summaries
    - **Domain annotations**: Each file/subdirectory tagged with its functional domain (Auth/User Data/API, etc.), enabling agents to filter out irrelevant files by domain
    - **Task Guide routing**: Predefined task-type → target-file mapping (e.g., "Add loss function → `losses.py`"), eliminating the agent's speculative semantic expansion
    - **Also Check cross-directory associations**: Empirically validated cross-directory file references in Task Guide (not exhaustive dependency graphs), ensuring related files are not missed
    
    ### Dependency Management (File-Level + Directory-Level)
    - **File Dependencies (within directory)**: IMPORT → EXPOSED_TO bidirectional table. Triggers additional reads ONLY when interface contracts need understanding or public signatures change — no unconditional chain-reading
    - **Cross-Dir Dependencies**: Each file lists its cross-directory Imports and Exposed To. Exposed To ≤5 lists exact paths; >5 degrades to a grep command ("foundational"), preventing stale static lists for heavily-depended-on files
    - **Directory-level Dependencies**: Marked internal/external to distinguish in-scope vs out-of-scope dependencies
    
    ### Large File Precision
    - **Feature Index (intent → line range)**: `.analysis.md` now includes intent-to-line-range mapping, allowing agents to match task keywords and jump directly to relevant code sections
    - **Logical Sections**: Fallback when no Feature Index row matches
    - **Top-Level Symbols table + Class Hierarchy**: Quick file structure overview
    
    ### Agent Behavior Constraints
    - **Two-Stage Read Protocol**: Stage 1 reads only Task Guide + Domain matched files; Stage 2 supplements only when analysis proves additional files are needed
    - **Dependency read gating**: Imports read only for interface contract understanding; Exposed To read only for public signature/semantics changes; >5 foundational files use grep + Domain filter first
    - **Auto-declared navigation protocol**: Writes 10 enhanced constraint rules + decision tree update rules into `CLAUDE.md` / `AGENTS.md`
    
    ### Engineering
    - **Parallel sub-agent generation**: Greedy bin-packing load balancing by code line count
    - **Dual mode**: Learning (one-time) / Maintenance (incremental via `git diff` + autonomous decision tree)
    - **Three-layer ignore rules**: Built-in defaults + `.gitignore` + user custom
    - **Multi-language output**: CODEMAP content language follows user's request language
    
    ## Generated Files
    
    After running this skill, the following files are added to the project:
    
    ```
    project-root/
    ├── CODEMAP.md                          # Root index (with Task Guide + Domain + Dependencies)
    ├── CLAUDE.md (appended navigation protocol)
    ├── src/
    │   ├── CODEMAP.md                      # src/ index (with Task Guide + Domain + Dependencies)
    │   ├── models/
    │   │   ├── CODEMAP.md                  # Files(Domain+Cross-Dir Deps) + File Deps + Task Guide
    │   │   └── large_model.py.analysis.md  # Large file analysis (Feature Index + Logical Sections)
    │   └── utils/
    │       └── CODEMAP.md
    └── tests/
        └── CODEMAP.md
    ```
    
    ## Installation
    
    Copy `SKILL.md` to your Claude Code skills directory:
    
    ```bash
    mkdir -p ~/.claude/skills/codemap
    cp SKILL.md ~/.claude/skills/codemap/SKILL.md
    ```
    
    ## Usage
    
    Trigger in a Claude Code session:
    
    - `/codemap`
    - Or say: "index this project" / "map this codebase" / "generate codemap"
    
    The skill will ask:
    1. Project mode (Learning / Maintenance)
    2. Enable parallel sub-agents? (default max: 3)
    3. Additional ignore patterns?
    
    Then it automatically scans and generates all CODEMAP.md and analysis files.
    
    ## Navigation Workflow
    
    How agents use CODEMAPs with the **constraint-based read protocol**:
    
    ```
    Step 1: Read root CODEMAP → Task Guide match against current task
            ↓ Hit → Target + Also Check directly define the initial file set
            ↓ Miss → Domain filter to identify target directories
    
    Step 2: Read target subdirectory CODEMAPs IN PARALLEL
            ↓ Task Guide for precise file targeting
            ↓ Domain column excludes non-matching files
            ↓ File Dependencies consulted ONLY conditionally (interface change / contract check)
    
    Step 3: Resolve cross-directory dependencies
            ↓ Also Check → read directly (empirically validated)
            ↓ Exposed To ≤5 → read only on signature/semantics change
            ↓ Exposed To >5 (foundational) → grep → Domain filter → read
    
    Step 4: Compile final target file set → deduplicate
    
    Step 5: Large files: read .analysis.md Feature Index first → locate line ranges
    
    Step 6: Batch-read ALL target source files IN PARALLEL (offset/limit for large files)
    
    Step 7 (Stage 2): Supplement only when analysis proves additional files are needed
    ```
    
    **Efficiency principle**: Reading 3-4 CODEMAPs (~200 lines) precisely locates 5-8 source files. Task Guide + Domain filter narrows the candidate set before any source code is read. Feature Index enables line-level precision for large files. Dependency information is a **safety net, not a reading mandate** — it triggers additional reads only under specific conditions.
    
    ## Update Rules (Maintenance Mode)
    
    After modifying code, the agent autonomously decides via this **decision tree**:
    
    1. **Structural change** (file/directory add/delete/move/rename) → Regenerate affected directory's full CODEMAP
    2. **Interface change** (public symbol signature/name changed) → Update Key Exports, Cross-Dir Dependencies (with threshold crossing handling), propagate upward
    3. **Implementation change** (bug fix / internal refactor / param tweak) → No CODEMAP update (only adjust Logical Sections line ranges if offset >20 lines in large files)
    
    ## License
    
    MIT
    diff --git a/codemap/SKILL.md b/codemap/SKILL.md
    index 73d729f..4de6f2f 100644
    --- a/codemap/SKILL.md
    +++ b/codemap/SKILL.md
    @@ -13,6 +13,16 @@ Generate hierarchical `CODEMAP.md` files that help agents locate relevant code w
     - Prefer positive guidance (Task Guide, Domain, Key Exports) over broad listings.
     - Dependencies are a safety net, not an invitation to chain-read.
     - Each CODEMAP describes only its own directory level. Child directory details belong in child CODEMAPs.
    +- Maintain a bijection between in-scope source entities and canonical inventory entries: no omissions, no duplicate ownership, no entries for nonexistent entities. Navigation references may be many-to-one; they are not additional inventory entries.
    +- Every code change requires considering addition, deletion, and modification together. Maintenance also actively retires superseded code within the authorized task scope; adding an index entry does not complete a migration.
    +
    +## Bijection and Freshness
    +
    +Define the inventory scope using the declared ignore rules. Every included immediate source file has exactly one canonical row in its directory's Files table; every included immediate child directory has exactly one Subdirectories row. Parent maps route to children without duplicating their inventories. CODEMAPs and companion analysis files are index artifacts, not source entities to recursively index. Key Exports and Task Guide are selective navigation views, not exhaustive inventories of all symbols or tasks.
    +
    +Validate both directions against the current source tree, including uncommitted changes: source → map catches omissions and duplicate ownership; map → source catches stale paths, symbols, responsibilities, dependencies, and line ranges. Do not exclude a source merely to hide an omission. Shared implementations can have multiple callers and navigation references without duplicate canonical entries.
    +
    +Freshness is a completion condition after each code change, not a timestamp update. In maintenance mode, inspect affected implementations for replacement, duplication, obsolete requirements, and completed migration scaffolding. Retire confirmed obsolete code and its exclusive tests, dependencies, build routes, documentation, and map entries in the same task. Retained compatibility paths need concrete consumers or a current contract and an explicit removal condition; recheck that condition when affected. Age or absence of local text references alone is not proof of obsolescence. Learning mode remains read-only for source code.
    
     ## Language Rule
    
    @@ -45,6 +55,12 @@ Merge in order:
    
     Include generated code only if it affects navigation; mark `Generated, do not edit manually`.
    
    +Bijection applies only within the declared inventory scope. Low-priority areas may be deliberately excluded or represented only by a directory boundary, according to the user's priorities and project conventions. Record these choices visibly in the root CODEMAP's **Scope and Exclusions** section; never silently omit them.
    +
    +Use `Path / pattern | Treatment | Reason / basis` rows. Treatment is either `Excluded` (no inventory obligation inside this scope) or `Boundary only` (retain one directory entry marked `Internals not indexed`, without a child map or internal file inventory). Patterns are root-relative; list any included exceptions explicitly. Record default exclusions and `.gitignore` as identifiable rule sources, grouping routine patterns rather than listing every ignored file. Project-specific omissions need concrete paths/patterns and reasons, not merely “defaults”. This section is authoritative; frontmatter `ignore` summarizes or points to it.
    +
    +Ignoring indexing does not mean code is obsolete, safe to delete, or exempt from correctness checks. If a task touches an excluded area or depends on its contract, inspect what is needed and reconsider the recorded scope. Update the scope decision if it changes; do not claim unindexed internals were verified.
    +
     ## Generation Workflow
    
     ### 1. Build Global Context
    @@ -81,7 +97,7 @@ Sub-agent prompts: self-contained, plain English. Include compressed global cont
    
     ### 4. Generate Per-Directory Maps
    
    -One `CODEMAP.md` per source directory (root + each subdirectory). Each map describes only the current directory level.
    +One `CODEMAP.md` per fully indexed source directory (root + included subdirectories). Excluded and boundary-only interiors do not require maps. Each map describes only the current directory level.
    
     ### 5. Assemble Root and Install Protocol
    
    @@ -113,6 +129,7 @@ Sections appear in this fixed order. Omit a section when it would be empty.
     | Section | Root | Mid-level | Leaf | Container |
     |---|---|---|---|---|
     | Summary | 1 sentence | 1 sentence | 1 sentence | 1 sentence |
    +| Scope and Exclusions | yes | — | — | root only |
     | Task Guide | yes | yes | yes | yes |
     | Subdirectories | yes | yes | — | yes |
     | Key Exports | yes | yes | if needed | — |
    @@ -120,6 +137,7 @@ Sections appear in this fixed order. Omit a section when it would be empty.
     | File Dependencies | — | immediate files only | yes | — |
    
     **Container directory** = directory with no source files, only subdirectories. Generate only Summary + Task Guide + Subdirectories.
    +Root containers also retain Scope and Exclusions, so deliberate omissions remain visible.
    
     ### Summary
    
    @@ -178,7 +196,7 @@ Rules:
     - One row per immediate file. **Never list files from child directories that have their own CODEMAP.**
     - `Domain` must match local Task Guide / Subdirectories values. `—` for trivial re-export files.
     - `Function`: one concise sentence. For large files, append `→ see <filename>.analysis.md`.
    -- Skip pure re-export `__init__.py` if exports are captured in Key Exports.
    +- Include pure re-export files such as `__init__.py` once in Files; Key Exports does not replace their canonical inventory row.
    
     ### File Dependencies
    
    @@ -292,10 +310,18 @@ The Deps column in Files tables marks cross-directory dependencies:
    
     ### Update Rules
    
    -After code changes, the agent autonomously evaluates:
    -- File/directory add, delete, move, rename → regenerate affected directory CODEMAP, update parent Subdirectories and Task Guide paths.
    -- Public symbol signature/return type change → update Key Exports and related Task Guide entries.
    -- Internal implementation change only → no update. Exception: update `.analysis.md` when Feature Index mapping becomes invalid.
    +Every code change requires evaluating all three operations within the affected task scope; perform each applicable operation without waiting for a separate cleanup request:
    +- Add: supply missing implementation or contract coverage and missing map entries for in-scope files/directories.
    +- Delete: remove confirmed redundant or superseded implementations and their exclusive tests, dependencies, build routes, documentation, and stale map/analysis entries. Check real consumers, supported platforms, dynamic/FFI entry points, and public contracts first.
    +- Modify: update existing implementations and canonical entries in place when responsibilities, interfaces, dependencies, paths, or analysis line ranges change. Moves/renames remove old routes and install current ones, including parent Task Guide/Subdirectories references.
    +
    +Completion requires a bijection between included source files/directories and their canonical local Files/Subdirectories entries: each entity appears exactly once, each entry resolves to a current entity, and excluded scope is explicit. Task Guide, Key Exports, and dependency links may reference the same entity; they must remain accurate without duplicating inventory ownership. Check source → map for missing/duplicate entries and map → source for stale entries and semantics, using the working tree rather than just the last commit.
    +
    +Deliberately unindexed areas must appear in root CODEMAP's Scope and Exclusions with root-relative paths/patterns, treatment (`Excluded` or `Boundary only`), and reason/basis. Boundary-only directories retain a marked directory entry, without an internal inventory obligation. Record rule sources for routine exclusions and explicit exceptions. When a task touches these areas or depends on their contracts, inspect the necessary code and reconsider scope; exclusion is neither deletion evidence nor a correctness exemption.
    +
    +Actively recheck retirement conditions for affected compatibility layers and migration code. Once replacement and consumer migration are verified, remove the old path in this task; retained exceptions need concrete consumers or a current contract and a removal condition. Do not accumulate `old/`, `v1/`, or `backup/` copies. Age or no local text references alone does not justify deletion.
    +
    +Pure internal equivalent changes may leave maps unchanged only after verifying that indexed responsibilities, contracts, dependencies, symbols, and line ranges remain accurate. Briefly report applicable additions/deletions/modifications, reasons for no action, and unresolved gaps; timestamps and passing builds alone do not establish map accuracy.
     ````
    
     ---
    diff --git a/project-maintenance/SKILL.md b/project-maintenance/SKILL.md
    index 8cc02dc..de60056 100644
    --- a/project-maintenance/SKILL.md
    +++ b/project-maintenance/SKILL.md
    @@ -17,13 +17,35 @@ description: Keep a project's code maps, implementations, tests and build routes
     Git 项目可先运行只读检查：
    
     ```bash
    -python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    +python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root> --json --max-paths 20
     ```
    
    -脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    +脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。限量输出仍保留完整计数和审查状态；分页或先只取计数的方式见[验证参考](references/verification.md)。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    +
    +## 工具与 agent 协作
    +
    +用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。已有格式的日常维护只读取命中的地图部分；首次建图或需要格式规则时再加载 `$codemap`。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
    +
    +先取计数、路径和行范围，再读必要内容；完整扫描结果可以留在工具侧。按任务需要选择 `rg`、AST、语言服务器或动态脚本，复用工具及维护状态入口。需要选型、批量核对或自建脚本时，读取[自动化与 codemap 协作](references/automation.md)。
    
     ## 随改动更新地图
    
    +### 双射性与增删改
    +
    +每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
    +
    +- **增**：补上当前契约所需的实现、有效验证和遗漏的地图条目。
    +- **删**：删除已确认冗余或退役的代码，以及其专属测试、依赖、构建路线、文档和地图条目。
    +- **改**：优先原地修正仍承担职责的实现，同步职责、接口、依赖、路径和行范围；不靠另加平行版本绕过旧问题。
    +
    +双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”，每个实体恰好归属一条；重导出文件也需纳入。任务入口、符号表和依赖边允许多处指向同一实体。采用 `$codemap` 时沿用其本层 Files/Subdirectories、父子路由及索引产物排除规则；其他地图保留等价归属，使用已有格式。
    +
    +以当前工作树双向核对：源码到地图查缺漏和重复归属，地图到源码查幽灵路径和失真的职责、接口、依赖及行范围。明确排除范围，不能通过扩大忽略规则掩盖缺漏。完成时简要说明增删改的处理或不适用依据，以及仍未闭合的缺口。
    +
    +双射只约束声明的索引范围。根地图统一记录排除项、仅保留边界的目录、原因及纳入例外；内部未索引的目录只保留唯一边界条目。采用 `$codemap` 时沿用其 Scope and Exclusions 格式，其他位置引用根地图的范围。
    +
    +忽略索引不等于过时、可删除或免于正确性验证。后续任务触及忽略区或依赖其契约时，读取所需代码并重新评估范围；决定变化时同步标记，不能把未索引内容宣称为已核对。维护脚本的扫描排除与地图的索引排除分别服务于变更检测和导航，不能未经判断直接互相套用。
    +
     使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    
     - 文件增删、移动、重命名：同步本层文件表、父目录路由及任务入口，移除已不存在目录的地图。
    @@ -43,6 +65,8 @@ python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root>
    
     ## 实现与测试一起退役
    
    +**时效淘汰性**：每次变更主动复查受影响旧实现和临时层的保留依据、迁移状态与移除条件。替代实现已验证、消费者已迁移或旧需求已撤销时，在本次任务内完成退役；不能仅在地图上标成 deprecated 后无限保留。暂时仍需保留的分支必须记录具体消费者或当前契约及可检查的移除条件，在后续相关改动时重新核对。时效由当前需求和迁移证据判定，不按文件年龄或任意期限机械删除。
    +
     对候选项先核对用途、调用者、构建目标、平台和公开契约，再在本次授权范围内处理：
    
     - 已替换并完成迁移的实现：清除旧代码、专属包装层、构建入口、依赖、文档、测试与 fixture；历史由 Git 保留，不复制 `old/`、`v1/`、`backup/` 源码树。
    ✓ • 0ms

## Activity

    $ pwd; git status --short; rg --files ../codemap .; wc -l ../codemap/SKILL.md ../codemap/README.md ../codemap/README_EN.md SKILL.md references/automation.md references/verification.md scripts/maintenance_state.py; cat SKILL.md references/automation.md references/verification.md agents/openai.yaml
    /home/fuurin/code/skills/project-maintenance
     M ../codemap/SKILL.md
     M SKILL.md
     M references/verification.md
     M scripts/maintenance_state.py
    ?? references/automation.md
    ../codemap/LICENSE
    ./references/automation.md
    ../codemap/README.md
    ../codemap/README_EN.md
    ../codemap/SKILL.md
    ./references/verification.md
    ./SKILL.md
    ./scripts/maintenance_state.py
    ./agents/openai.yaml
      337 ../codemap/SKILL.md
      122 ../codemap/README.md
      122 ../codemap/README_EN.md
       96 SKILL.md
       52 references/automation.md
       53 references/verification.md
      301 scripts/maintenance_state.py
     1083 total
    ---
    name: project-maintenance
    description: Keep a project's code maps, implementations, tests and build routes current after feature work, refactoring, API changes or backend migrations. Use for maintenance and retirement of superseded code in assembly, Rust, Python, TypeScript, C and C++ projects, especially native UI and dedicated inference engines.
    ---
    
    # Project Maintenance
    
    把维护作为当前开发任务的完成条件。同步受影响的地图，清除已被替代的实现和失效测试，验证仍然承诺的产品行为。对用户已经授权的任务内维护直接执行；跨出任务范围、改变支持平台或对外接口时，先给出具体影响。
    
    ## 确认当前契约
    
    - 从指定项目根目录开始，读取 `AGENTS.md`、当前需求及已有地图。保留规定的语言、渲染后端、推理引擎、平台支持和源码所有权边界。
    - 检查已提交、暂存、未暂存及新增文件。区分本次任务和其他人的未完成工作，不替别人整理不相关改动。
    - 新项目随首个可运行实现建立地图；已有项目以当前源码验证地图，不以旧文档推断不存在的接口。
    - 链接进 UI 仓库的引擎源码、子模块和独立 checkout 各自验证真实目录；一个仓库的记录不证明另一个源码树已经同步。
    
    Git 项目可先运行只读检查：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root> --json --max-paths 20
    ```
    
    脚本比较上次审查记录与当前文件内容，包括测试、配置、文档、地图和未提交改动；首次运行给出待建立基线的清单。限量输出仍保留完整计数和审查状态；分页或先只取计数的方式见[验证参考](references/verification.md)。它不修改源码，不判断某个测试或函数应该删除。没有 Git 时按显式目录做同样的人工审查；仅在项目创建任务已允许初始化仓库时建立 Git。
    
    ## 工具与 agent 协作
    
    用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。已有格式的日常维护只读取命中的地图部分；首次建图或需要格式规则时再加载 `$codemap`。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
    
    先取计数、路径和行范围，再读必要内容；完整扫描结果可以留在工具侧。按任务需要选择 `rg`、AST、语言服务器或动态脚本，复用工具及维护状态入口。需要选型、批量核对或自建脚本时，读取[自动化与 codemap 协作](references/automation.md)。
    
    ## 随改动更新地图
    
    ### 双射性与增删改
    
    每次代码变更都必须评估 **增、删、改**，按实际需要执行；“更新、完善项目”已经包含任务范围内的补缺、去冗余和修改旧实现，不必等待用户另提清理要求，也不要求为凑齐三类而制造改动。
    
    - **增**：补上当前契约所需的实现、有效验证和遗漏的地图条目。
    - **删**：删除已确认冗余或退役的代码，以及其专属测试、依赖、构建路线、文档和地图条目。
    - **改**：优先原地修正仍承担职责的实现，同步职责、接口、依赖、路径和行范围；不靠另加平行版本绕过旧问题。
    
    双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”，每个实体恰好归属一条；重导出文件也需纳入。任务入口、符号表和依赖边允许多处指向同一实体。采用 `$codemap` 时沿用其本层 Files/Subdirectories、父子路由及索引产物排除规则；其他地图保留等价归属，使用已有格式。
    
    以当前工作树双向核对：源码到地图查缺漏和重复归属，地图到源码查幽灵路径和失真的职责、接口、依赖及行范围。明确排除范围，不能通过扩大忽略规则掩盖缺漏。完成时简要说明增删改的处理或不适用依据，以及仍未闭合的缺口。
    
    双射只约束声明的索引范围。根地图统一记录排除项、仅保留边界的目录、原因及纳入例外；内部未索引的目录只保留唯一边界条目。采用 `$codemap` 时沿用其 Scope and Exclusions 格式，其他位置引用根地图的范围。
    
    忽略索引不等于过时、可删除或免于正确性验证。后续任务触及忽略区或依赖其契约时，读取所需代码并重新评估范围；决定变化时同步标记，不能把未索引内容宣称为已核对。维护脚本的扫描排除与地图的索引排除分别服务于变更检测和导航，不能未经判断直接互相套用。
    
    使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    
    - 文件增删、移动、重命名：同步本层文件表、父目录路由及任务入口，移除已不存在目录的地图。
    - 接口或依赖变化：同步导出符号、调用入口、跨目录依赖及消费者。
    - 汇编与 FFI：记录真实符号、调用约定、结构布局和偏移、内存所有权、回调线程，以及 C/Rust 包装层；用头文件、声明、调用点和链接符号核对。
    - UI 与推理引擎：记录事件到状态到渲染/推理的路径；模型边界写明 tensor shape、dtype、布局、量化格式和必要的转换。
    - 实现变化：若职责、数据流、契约或大文件功能行范围变化，仍然更新相关说明；纯内部等价修改可以不改地图，并说明判断依据。
    
    地图保留一份当前版本，原地更新。只在 agent 规则文件中放简短入口和完成条件，避免复制整份地图。
    
    ## 文档一起更新和退役
    
    - 代码、接口、命令、目录、配置、支持平台或用户流程变化时，检查 README、架构说明、运行手册、示例、任务入口和交叉链接；把仍然有效的说明改成当前行为。
    - 功能、接口或构建路线退役时，删除只服务于它的文档、示例、截图、fixture、导航条目和失效链接；先核对反向链接与仍在使用的共享资源。Git 历史承担已删除内容的留存。
    - 保留一份权威说明，合并重复文档，避免用过时的 `old/`、`v1/`、`backup/` 文档树掩盖当前状态。文档删除必须有当前契约或调用关系依据，不能只因文件很旧或无人引用。
    - 完成维护后，至少检查受影响 Markdown 链接、命令、路径和状态描述；不能验证的外部链接或平台行为要明确记录。
    
    ## 实现与测试一起退役
    
    **时效淘汰性**：每次变更主动复查受影响旧实现和临时层的保留依据、迁移状态与移除条件。替代实现已验证、消费者已迁移或旧需求已撤销时，在本次任务内完成退役；不能仅在地图上标成 deprecated 后无限保留。暂时仍需保留的分支必须记录具体消费者或当前契约及可检查的移除条件，在后续相关改动时重新核对。时效由当前需求和迁移证据判定，不按文件年龄或任意期限机械删除。
    
    对候选项先核对用途、调用者、构建目标、平台和公开契约，再在本次授权范围内处理：
    
    - 已替换并完成迁移的实现：清除旧代码、专属包装层、构建入口、依赖、文档、测试与 fixture；历史由 Git 保留，不复制 `old/`、`v1/`、`backup/` 源码树。
    - 临时适配层：迁移完成就移除；仍有使用者时，写清当前消费者和具体移除条件。
    - 重复或仅验证 mock 自身的测试：合并或删除，保留不同输入、平台、边界条件和已发现故障的有效断言。
    - 测试取消或改变的需求：按当前契约退役或重写，同时核对相关 fixture、runner 配置和文档。
    - 失败、被跳过、年代久远、覆盖率低或图中没有入边，都不是单独的删除依据。先区分产品回归、环境问题、有效平台测试和真正失效的需求。
    - 新测试只验证本次行为、接口或实际故障，避免把当前实现细节再写一遍。对仍有效的回归保持独立检测能力；不能为使检查通过而删除断言、改宽误差或重录错误的快照。
    
    外部导出、动态注册、函数指针、FFI、汇编入口和跨平台分支需要额外核对消费者。只证明“无本地文本引用”时，报告剩余不确定性，继续完成其余可确认的维护。
    
    ## 验证并记录当前状态
    
    按 [验证与退役判据](references/verification.md) 只选择本次涉及的语言、平台、UI 或推理验证。优先复用现有验证入口；不要为了使用这个 skill 重建一套重复测试框架。
    
    完成本次维护和有效验证后，在原地替换单份审查记录：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py record --root <project-root> --reviewed
    python3 <skill-dir>/scripts/maintenance_state.py check --root <project-root>
    ```
    
    `--reviewed` 表示 agent 已实际审查当前范围并完成应做的验证，不是用户批准，也不能替代验证。未完成的关联改动、未知失败或未验证的其他源码树必须如实报告；不能为让检查变绿而重录。
    
    记录位于 `.project-maintenance/state.json`，包含当前文件哈希与审查版本。`check` 在文件或审查范围变化后返回非零，供已有验证命令、pre-commit 或 CI 调用；新增这些集成时使用项目现有方式，编辑一个可重复更新的入口。脚本的范围、排除模式和能力限制见验证参考。
    
    最终报告本次同步的地图、退役项及其依据、实际验证结果和未解决问题。指标是当前契约和可维护性，不是删除数量、测试数量或通过率。
    # 自动化与 codemap 协作
    
    需要选择工具、批量核对或动态生成辅助脚本时读本参考。工具提取可重复的事实，agent 判断当前契约与变更影响；读取范围随证据扩展。本文描述两种技能的协作，地图格式与导航规则由可用的 `$codemap` 或项目现有地图维护。
    
    ## 已有能力及归属
    
    | 能力 | 复用入口 | project-maintenance 的工作 |
    | --- | --- | --- |
    | 任务路由、Domain 过滤、条件读取消费者 | codemap 的 Task Guide、Files、Dependencies | 用变更证据选择入口，判断公开契约是否受影响 |
    | 大文件功能到行范围的定位 | codemap 的 `.analysis.md` Feature Index | 核对受影响范围仍对应当前源码 |
    | 文件/目录唯一归属、范围和排除项 | codemap 的 Files、Subdirectories、根 Scope and Exclusions | 检查当前工作树的缺漏、重复、幽灵路径并同步 |
    | 地图增删改与父子路由更新 | codemap 维护模式 | 将地图更新纳入实现、文档、构建和验证的完成条件 |
    | 审查之后发生了哪些变化 | `maintenance_state.py` 的 inspect/check | 结合 Git 差异区分本次任务和其他改动 |
    | 旧实现、测试和迁移层是否应退役 | 当前源码、消费者、产品契约及验证入口 | agent 判断保留依据与移除条件，执行获授权的维护 |
    
    维护状态的扫描范围与地图的索引范围分别保存在审查记录和根地图中；扫描会包含许多无需索引的配置、fixture 和文档。临时候选清单用于本次操作，不成为另一份权威地图。采用 codemap 时使用其维护模式；普通维护沿用既有配置，只有新建或改变索引范围时才处理相应配置问题。
    
    ## 按需求选择工具
    
    | 要解决的问题 | 工具与使用条件 | 返回给 agent 的信息 |
    | --- | --- | --- |
    | 找文件及本次差异 | Git、`rg --files`；审查基线用已有状态脚本 | 状态、数量、路径、差异片段 |
    | 找名称、路径和文本消费者 | `rg -l` 先收候选，`rg -n -F` 再定位；按目录/类型缩小范围 | 去重路径及必要的匹配行 |
    | 找语法结构、批量迁移同类调用 | 有需要时使用 ast-grep 的结构搜索、outline、重写预览 | 符号/节点、文件、范围、替换差异 |
    | 分辨同名符号、查询引用、语义重命名 | 项目已有的语言服务器，例如 clangd、rust-analyzer；配置、feature 和索引覆盖需有效 | 真实符号的定义与引用位置 |
    | 跨语言声明定位，缺少现成结构提取器 | 已安装的 Universal Ctags；Python 可直接用标准库 AST | 符号及行号，作为导航候选 |
    | 地图表格与当前目录的批量集合核对 | 先复用 codemap/项目工具；缺少时生成限定格式的临时比较脚本 | 缺漏、重复归属、幽灵路径、越界行范围 |
    | 重复全量提取确实耗时 | 优先复用语言服务器缓存；有测量依据再考虑 SQLite 存放按内容哈希更新的事实 | 与本次变更相关的行，标明来源和有效版本 |
    
    ast-grep 支持结构搜索、JSON 输出和重写；当前官方版本还提供 outline。使用前检查本地 `--help` 的能力，缺少工具时先选已安装且足够的方式。[官方 CLI](https://ast-grep.github.io/reference/cli.html) 与[工具指南](https://ast-grep.github.io/guide/tooling-overview.html)。Universal Ctags 的 JSON 支持取决于编译选项，其符号清单用于定位。[官方 JSON 文档](https://docs.ctags.io/en/latest/man/ctags-json-output.5.html)。clangd 的后台索引提供项目引用信息，当前打开文件的动态索引用于跟随编辑变化。[索引设计](https://clangd.llvm.org/design/indexing)。
    
    结构匹配和文本命中都只是证据。无命中时核对搜索忽略项、动态注册、FFI、导出接口和有效平台分支；单凭它们不执行删除。语言服务器也只覆盖其有效配置下的源码。源码内容变化后，旧行号、片段和提取结果需要重新核对。
    
    ## 一次维护怎样执行
    
    1. **取增量事实**：用 `inspect --json --max-paths 20` 取得完整计数和首批路径，必要时分页。首次没有审查记录时，清单表示待建立基线，不能把全部 added 视为本次新增；结合 Git 的暂存、未暂存差异与未跟踪文件判断。
    2. **通过地图定向**：沿 codemap 的任务路由进入受影响目录，用 Feature Index 确定大文件行范围。有具体契约缺口再查消费者，消费者还可能位于本次未修改的目录。
    3. **取得最小证据**：批量运行独立查询，工具侧去重和筛选；返回路径、符号、行号和所需片段。完整清单较大时保存在临时文件并明示截断，读取后续项或按任务筛选，不能用一个 `head` 隐藏未知范围。
    4. **agent 判断并执行增删改**：以实现、消费者和当前契约决定操作。适用的统一替换先看匹配与差异预览，再执行受限的批量编辑；任意字符串替换无法代替语义重命名。
    5. **闭合影响**：同步 codemap、相关说明、构建路线和有效验证。完成实际审查后才 record，再 check；分页参数只压缩报告，不缩小记录范围。
    
    例如先用 `rg -l -F -e 'OldEngine' src tests` 取得候选路径；再对选定文件定位行段。具体命中需要 agent 判断，不能把测试字符串、注释和实际调用混为一个消费者。Git 与 rg 的文件过滤结果也不能直接充当地图的声明范围。
    
    ## 动态自建工具的条件
    
    当既有命令的组合已经够用时直接组合。有明确的重复计算或格式转换需求时，再写一个窄小辅助脚本。例如 codemap 格式的 Files/Subdirectories 双向集合比较应复用或补到 codemap 的工具归属；维护状态继续使用本技能的单一脚本入口。
    
    临时脚本放在获允许的临时目录或项目既有本地工具目录。参数明确给出根目录、输入、声明的范围、格式和输出位置；未知表格格式要报“未解析”，不能假装核对成功。只比较权威清单条目，避免把 Task Guide、Key Exports 和导航链接算作重复归属。范围、例外和仅保留边界需依据根地图解释，不能照搬维护扫描的排除项。
    
    把事实与候选判断分开输出，保留来源路径、当前内容标识和必要行号；输出包括完整计数、返回数量、遗漏/截断、后续读取入口和错误。解析、读取或子进程失败时返回失败；参数化调用工具，避免将仓库文本拼成可执行的 shell 命令。批量写入使用具体编辑清单或补丁，并核对预期内容仍匹配当前文件。
    
    只用于一次任务的脚本无需长期维护。经实际复用证明有收益的脚本再归入对应 skill/项目工具，并补充使用说明和行为验证；保留一套接口。节省 token 主要靠减少返回的无关内容和重复读取；多 agent 会增加总调用量，只有已获授权且任务能独立分工时，才用它隔离中间上下文。
    # 验证与退役判据
    
    只读取本次涉及的部分。使用项目已有命令、锁定版本、功能组合和支持平台，先记录已存在的失败，再核对本次差异。缺少完成验证所必需的工具或数据时补齐并执行；不能把未验证写成通过。
    
    ## 语言与接口边界
    
    | 场景 | 需要的证据 |
    | --- | --- |
    | 汇编、C、C++ | 用实际目标和编译选项构建并链接；核对导出/引用符号、ABI、栈与缓冲区对齐、寄存器保存、布局断言及有效的平台分支。删汇编符号前核对 C/Rust 声明、链接脚本、动态查找和函数指针注册。 |
    | Rust / FFI | 当前 feature 与 target 下的构建和接口测试；对应安全契约、布局、所有权及异常边界。Miri 可验证 Rust 侧契约，但不能证明任意 C/汇编实际调用正确。 |
    | Python | 当前环境的类型、lint 和相关功能验证；核对动态导入、插件注册、CLI、模型导出/转换入口及包的公开接口。 |
    | TypeScript | 当前配置的类型检查、构建与相关行为测试；核对动态加载、路由、注册表、包导出及原生桥接。死代码工具只提供候选。 |
    
    ## 原生与 Web UI
    
    - 原生 UI：运行实际窗口，验证此次涉及的输入、焦点、文本、缩放、布局、渲染路径及与真实引擎的交互；检查截图/帧与运行日志。构建成功和生成一张图不证明交互完整。
    - Web UI：在真实浏览器中验证此次流程、控制台、网络和相应视口；使用当前页面状态定位元素。纯 DOM 或组件快照不证明连接实际后端后的流程。
    - 视觉基线只在需求确认改变且实际画面核对后更新。移除弃用界面的快照与 fixture 时，同时核对共用资源的消费者。
    
    ## 专用推理框架
    
    - 从原始模型或已验证的参考实现建立输入与输出契约：shape、dtype、stride/layout、量化参数、采样/帧约定以及适用精度。
    - 按实际工作负载检查数值对齐和完整推理路径。新内核需要验证尾部尺寸、对齐、有效 ISA 分派和 fallback；“更快”不证明“相同”。
    - 性能采用同机、同负载、同契约的稳定比较，记录端到端与真实热点。复用已有 `performance-gradient-optimization` 或项目等效方法。
    - 保留当前对齐/性能基线的明确用途；淘汰无用的候选实现、重复基准入口和失效数据。基线输入、期望结果与硬件/模型来源需要可追溯，历史结果交由项目既有存档机制保存。
    
    ## 持续维护检查脚本
    
    `scripts/maintenance_state.py` 仅依赖 Python 标准库和 Git，支持包括尚无首次提交的 Git 工作树：
    
    ```bash
    python3 <skill-dir>/scripts/maintenance_state.py inspect --root . --json
    python3 <skill-dir>/scripts/maintenance_state.py record --root . --reviewed
    python3 <skill-dir>/scripts/maintenance_state.py check --root .
    ```
    
    agent 首次读取可用 `inspect --root . --json --max-paths 20`；只取计数用 `--max-paths 0`。限量 JSON 的 `change_counts` 和 `separate_checkout_count` 始终覆盖完整扫描，`changes` 与 `separate_checkouts` 只包含本页。分页顺序为地图的增/改/删、其他文件的增/改/删、独立 checkout，各组按路径排序。
    
    按 `pagination.next_offset` 继续传入 `--offset`，并核对各页 `report_id` 一致；内容、审查记录或范围变化时重新取页。`pagination.total/returned/omitted/truncated` 明示完整量与本页量；计数模式不提供下一页，应改用正数限量。分页限制路径条目数量，路径长度和范围元数据仍会影响输出大小。省略 `--max-paths` 恢复原有完整 JSON；文本输出仍使用原有摘要。
    
    输出限量不改变扫描范围、审查记录或 `check` 退出码。不要把本页空列表、最后一页或截断结果当成无变更；以完整状态、计数和实际审查为准。`--root` 会解析为 Git 顶层，单个包的任务应再按地图与任务路径筛选，不能把仓库级记录说成仅审查了当前包。
    
    - 扫描 Git 跟踪文件及未被忽略的新增文件，比较实际内容哈希；检测修改、增删、重命名及地图本身的变化。模式或时间戳相同的内容修改仍会触发。
    - tracked fixture、lockfile、配置与文档也在范围内。`CODEMAP.md`、`codemap.md` 和 `*.analysis.md` 在报告中单列；记录文件自身不进入指纹。
    - 默认排除缓存/依赖目录，以及根目录的 `build/`、`dist/`、`out/`、`coverage/`。若项目把真实源码放在这些位置，通过 `--include-generated` 纳入；该选择会记录在范围中。
    - 大模型、数据集或受项目明确排除的产物，可使用根目录相对的 `--exclude 'weights/**'` 等模式；目录模式如 `datasets/` 排除其子树。模式影响扫描范围，不能拿它掩盖未完成维护。设置会保存在审查记录中，后续检查复用；显式重新提供模式会替换已保存的自定义排除。
    - 例如：`inspect --root . --exclude 'weights/**'`；确认范围后以相同选项运行 `record --reviewed`。恢复自定义排除为空使用 `--clear-excludes`。修改排除范围会要求新的审查记录。
    - 符号链接只记录链接目标，不跟随读取外部文件；子模块只记录 checkout 状态。报告中的 `separate_checkouts` 必须在其真实源码树单独审查和检查，链接本身不变不代表引擎内容不变。
    - `record` 只原子替换 `.project-maintenance/state.json`。若使用 CI，可把这份记录交给 Git；后续在同一路径更新，不生成每次任务的副本。
    - `inspect` 只读并返回清单。`check` 返回 `0` 表示该范围与已审查指纹一致，`1` 表示需审查或尚无记录，`2` 表示仓库、读取或记录错误；错误不能当作干净状态。
    - 一致的指纹只证明“之后没有新的受监测文件变化”，不证明地图语义、死代码判断或产品正确性。实际调用者和有意义的验证仍由 agent 核对。
    
    在已有项目中接入检查脚本时使用该 skill 的实际路径或项目自己的固定入口。不要假定其他机器安装在同一个 home 路径，也不要把 skill 私有源码大量复制进项目。
    interface:
      display_name: "Project Maintenance"
      short_description: "Keep code maps, active code and tests current"
      default_prompt: "Use $project-maintenance to finish this change by updating code maps, retiring obsolete code and tests, and verifying the current product."
    ✓ • 0ms

## Assistant

主要重复集中在 codemap 的生成格式、两份导航协议，以及 project-maintenance 中重复的地图规则。我会把格式和协议移到按需读取的参考里，日常入口只保留必要的决策步骤；同时修正中英文 README 的旧规则，工具选型保留 Git、rg、ast-grep 和语言服务器。

## Activity

    $ for instruction_file in ../AGENTS.md ../codemap/AGENTS.md ../codemap/references/AGENTS.md AGENTS.md references/AGENTS.md scripts/AGENTS.md; do if [ -f "$instruction_file" ]; then cat "$instruction_file"; fi; done; sed -n '1,155p' /home/fuurin/.codex/skills/.system/skill-creator/SKILL.md; cat ../codemap/README.md; sed -n '1,115p' ../codemap/SKILL.md
    ---
    name: skill-creator
    description: Create or update a Codex skill with appropriately scoped instructions and any needed supporting resources.
    metadata:
      short-description: Create or update a skill
    ---
    
    # Skill Creator
    
    Create skills that give Codex useful, non-obvious guidance without constraining unrelated work.
    
    ## Core Principles
    
    **Assume Codex is already capable.** Include only information that changes its decisions or improves its work. Remove generic advice, repeated instructions, speculative edge cases, and examples that do not materially clarify the task.
    
    **Preserve user intent and scope.** A skill should support the requested task, not replace the user's chosen product, expand the assignment, modify unrelated configuration, or imply permission for additional external actions. Do not turn a particular example, past failure, or personal preference into a universal requirement.
    
    Approval to complete a task does not expand its scope or execution permissions. For retrying or externally mutating workflows, define a stopping condition proportional to the risk.
    
    **Match specificity to the risk.** Give the model room to choose an appropriate approach when multiple approaches are reasonable. Use detailed steps, deterministic scripts, or absolute language only when correctness, safety, permissions, or a genuinely fragile workflow requires them.
    
    For open-ended work, describe the outcome and relevant decision criteria. For workflows with a preferred shape, offer useful examples or configurable scripts. Reserve fixed sequences and narrow parameters for operations where deviation would cause a concrete problem. Preserve non-obvious operational invariants, distinguish actual requirements from optional recommendations or local conventions, and avoid restating policies already enforced elsewhere.
    
    **Keep discovery cheap and precise.** Skill names and descriptions are available before a skill is loaded. Describe the actual capability and when it applies, adding exclusions only when they prevent likely misrouting. Avoid exhaustive capability lists and catchalls that attract unrelated requests.
    
    Keep skills self-contained; refer to another skill or tool only when the requested workflow genuinely requires it and it is available in the target environment. Specialized review, hardening, or audit workflows should apply when requested or genuinely needed, not merely because ordinary work touches the same subject.
    
    **Disclose detail progressively.** Keep shared purpose, essential constraints, and useful routing in `SKILL.md`. Put substantial mode-specific guidance, schemas, examples, or procedures in supporting references and read only the references relevant to the current task. A simple self-contained skill does not need a router or extra files.
    
    ## Anatomy of a Skill
    
    Every skill is a folder containing a required `SKILL.md` file and any optional resources its actual workflow needs:
    
    ```text
    skill-name/
    |-- SKILL.md                 Required skill instructions
    |   |-- YAML frontmatter     Required name and description
    |   `-- Markdown body        Instructions loaded when the skill is used
    |-- agents/                  Optional UI metadata and invocation policy
    |   `-- openai.yaml
    |-- scripts/                 Optional executable helpers
    |-- references/              Optional documentation loaded as needed
    `-- assets/                  Optional files used in generated output
    ```
    
    Choose the structure that fits the actual task. Some skills are short and self-contained; others route among operating modes or delegate complex mechanics to scripts. Avoid creating directories, placeholders, examples, or ancillary documentation without a clear use.
    
    ### SKILL.md
    
    The YAML frontmatter identifies the skill and determines when it should be considered. Include the required `name` and `description`, and preserve supported optional fields such as existing `metadata` when appropriate.
    
    The Markdown body is loaded only when the skill is used. Put the purpose, essential workflow, real constraints, and useful links there. Keep detailed procedures and examples in supporting references when they are relevant only to particular modes.
    
    Skill information is disclosed in three stages:
    
    1. **Name and description:** Available during skill selection, so keep them concise and discriminating.
    2. **SKILL.md body:** Loaded when the skill applies, so keep its instructions relevant to that task.
    3. **Supporting resources:** Read or execute only when the current task actually needs them.
    
    The entrypoint should be as short as the task permits while retaining important constraints. A large upper bound is not a target: move conditional detail into references when doing so improves clarity or context use, rather than waiting for the file to become unwieldy.
    
    ### Scripts
    
    Use `scripts/` for executable code when the same logic would otherwise be rewritten repeatedly or deterministic execution materially improves reliability.
    
    - **Example:** `scripts/rotate_pdf.py` for a PDF operation that would otherwise require recreating the same code.
    - **Useful for:** Repeated transformations, reliable API operations, data processing, and other concrete automation.
    - **Validation:** Run new or changed scripts to verify their behavior. Scripts can usually be executed without loading their full implementation into context, although an agent may need to inspect them when patching or adapting them.
    
    ### References
    
    Use `references/` for documentation that is needed only in particular contexts.
    
    - **Examples:** `references/schema.md` for database tables, `references/policies.md` for domain rules, `references/api_docs.md` for an API, or separate writing guides for different deliverables.
    - **Useful for:** Schemas, API documentation, company policies, format-specific procedures, detailed workflows, and substantial examples.
    - **Routing:** Link each reference from `SKILL.md` or another relevant resource and explain when it should be read. Keep information in one place instead of duplicating it across the entrypoint and references.
    
    Keep references focused on maintained, task-specific information that changes the agent's decisions. Avoid copied manuals, exhaustive catalogs, and generic tutorials already available from authoritative sources. Before removing existing resources, inspect their callers and purpose.
    
    For large references, include useful search terms or a short contents section when that makes the needed material easier to find.
    
    ### Assets
    
    Use `assets/` for files that belong in generated output rather than in the model's instructions.
    
    - **Examples:** `assets/logo.png`, `assets/slides.pptx`, `assets/font.ttf`, or `assets/frontend-template/`.
    - **Useful for:** Templates, images, fonts, icons, boilerplate projects, and other files copied or adapted into the result.
    - **Context:** Do not load assets as instructions unless the task requires inspecting them.
    
    ### UI Metadata and Invocation Policy
    
    `agents/openai.yaml` can provide UI-facing metadata such as `display_name`, `short_description`, and `default_prompt`, along with invocation policy. When creating or updating those settings, read [references/openai_yaml.md](references/openai_yaml.md) and keep the values consistent with the skill.
    
    Automatic skill selection is allowed by default. Change that default only when the user explicitly requests an explicit-only skill:
    
    ```yaml
    policy:
      allow_implicit_invocation: false
    ```
    
    This keeps the skill available when explicitly invoked as `$skill-name` without adding it to the model context automatically. Preserve unrelated existing UI, policy, and dependency fields when updating `agents/openai.yaml`.
    
    The initializer creates this file automatically. For new or interface-only metadata, generate it with:
    
    ```bash
    scripts/generate_openai_yaml.py <path/to/skill-folder> --interface key=value
    ```
    
    The generator replaces the entire file. If an existing file contains `policy` or `dependencies`, update only the intended fields in place instead of regenerating it.
    
    Include optional interface fields only when the user provides or requests them.
    
    ### What Not to Include
    
    Include files that directly support the skill's work. Avoid adding a `README.md`, installation guide, changelog, duplicated quick reference, or other auxiliary documentation unless a specific task or packaging requirement calls for it.
    
    ## Progressive Disclosure in Practice
    
    For a skill with multiple substantial modes, keep the shared guidance and mode-selection criteria in `SKILL.md`. Link each supporting reference where its use becomes relevant. Do not load every reference by default, duplicate reference content in the entrypoint, or add a routing layer when there is nothing meaningful to route.
    
    For example, a deployment skill can keep provider selection in `SKILL.md` and separate provider details:
    
    ```text
    cloud-deploy/
    |-- SKILL.md
    `-- references/
        |-- aws.md
        |-- gcp.md
        `-- azure.md
    ```
    
    When the user chooses AWS, read `references/aws.md`; do not also load the GCP and Azure guides. The same pattern can separate business domains, deliverable types, or other genuinely distinct operating modes.
    
    A short skill can instead route to details only when an advanced operation needs them:
    
    ```markdown
    ## Documents
    
    Handle ordinary edits directly.
    
    - For tracked changes, read [references/redlining.md](references/redlining.md).
    - For document internals, read [references/ooxml.md](references/ooxml.md).
    ```
    
    These examples illustrate options, not a required structure. Choose the organization that makes the skill easier to use without loading irrelevant material.
    
    ## Create or Update a Skill
    
    Adapt the work to the request. Creating a complex new skill may involve understanding realistic use cases, choosing supporting resources, initializing files, writing instructions, and validating the result. A narrow update to an existing skill may require only a focused edit and validation.
    
    Ask clarifying questions only when the missing information matters and cannot be reasonably inferred. Respect a user-specified location; otherwise create discoverable skills in `$CODEX_HOME/skills`, or `~/.codex/skills` when `CODEX_HOME` is unset.
    
    Keep automatic skill selection enabled unless the user explicitly requests an explicit-only skill. When the intended invocation mode is genuinely unclear and matters to the requested workflow, ask whether the user wants normal automatic discovery or explicit-only invocation; otherwise preserve the default. Do not infer explicit-only invocation from sensitive operations or required approvals: keep the skill discoverable and require authorization immediately before the actual mutation. Preserve an existing skill's invocation policy unless the user asks to change it.
    
    For a new or substantially revised skill, consider the actual requests it should handle and which reusable resources would improve those tasks:
    # codemap-skill
    
    **中文** | **[English](README_EN.md)**
    
    ---
    
    为 Claude Code 及其他 AI 编程 Agent 设计的**层级化代码库导航索引** Skill。
    
    在项目根目录和每个源码子目录下生成 `CODEMAP.md` 索引文件，以及针对超大文件（>1000 行）的 `<filename>.analysis.md` 深度分析伴生文件。采用 Task Guide → Domain 过滤 → 依赖安全网的**约束式导航策略**，从源头防止 Agent 扩散性读取无关文件。
    
    ## 核心特性
    
    ### 索引与导航
    - **层级化索引**：根目录 + 每个子目录各自生成 `CODEMAP.md`，包含精简目录结构、文件/子目录功能概要
    - **Domain 功能域标注**：每个文件/子目录标注所属功能域（Auth/User Data/API 等），Agent 按域过滤排除无关文件
    - **Task Guide 任务路由**：预定义任务类型→目标文件的精确映射（如"新增 Loss 函数 → `losses.py`"），消除 Agent 自行语义扩展的不确定性
    - **Also Check 跨目录关联**：Task Guide 中附带经验性跨目录关联文件列表（非全量依赖图），确保关联文件不被遗漏
    
    ### 依赖关系管理（文件级 + 目录级）
    - **File Dependencies（同目录内）**：IMPORT → EXPOSED_TO 双向链表，仅在接口契约需要理解或公开签名变更时触发读取，禁止无条件链式遍历
    - **Cross-Dir Dependencies（跨目录）**：每个文件标注其跨目录 Imports 和 Exposed To。Exposed To ≤5 精确列出文件路径；>5 则降级为 grep 动态查询指令（"foundational"），防止基础文件的静态列表过时
    - **目录级 Dependencies**：标注 internal/external 以区分视野范围内外的依赖
    
    ### 大文件精确定位
    - **Feature Index（功能→行范围索引）**：`.analysis.md` 中新增意图→行范围映射表，Agent 直接匹配任务关键词定位需要读取的代码段
    - **Logical Sections（逻辑分段）**：作为 Feature Index 未匹配时的回退方案
    - **顶层符号表 + 类继承关系**：快速了解文件结构
    
    ### Agent 行为约束
    - **Two-Stage Read Protocol**：Stage 1 仅读取 Task Guide + Domain 匹配的文件；Stage 2 仅在分析证明确实需要时补充读取
    - **依赖读取条件门控**：Imports 仅在接口契约理解需要时读；Exposed To 仅在公开签名/语义变更时读；>5 foundational 文件先 grep 再按 Domain 过滤
    - **自动写入导航协议**：在 `CLAUDE.md` / `AGENTS.md` 中写入 10 条增强约束规则 + 决策树更新规则
    
    ### 工程特性
    - **并行 Sub-agent 生成**：按代码行数贪心装箱均衡负载
    - **双模式**：学习模式（一次生成）/ 维护模式（基于 `git diff` 增量更新 + 决策树自主判断更新范围）
    - **三层忽略规则**：内置默认 + `.gitignore` + 用户自定义
    - **多语言输出**：CODEMAP 内容语言跟随用户提问语言
    
    ## 生成的文件
    
    运行此 Skill 后，项目中会新增以下文件：
    
    ```
    project-root/
    ├── CODEMAP.md                          # 根目录索引（含 Task Guide + Domain + Dependencies）
    ├── CLAUDE.md (追加导航协议)             # 或 AGENTS.md
    ├── src/
    │   ├── CODEMAP.md                      # src/ 索引（含 Task Guide + Domain + Dependencies）
    │   ├── models/
    │   │   ├── CODEMAP.md                  # 含 Files(Domain+Cross-Dir Deps) + File Deps + Task Guide
    │   │   └── large_model.py.analysis.md  # 大文件分析（含 Feature Index + Logical Sections）
    │   └── utils/
    │       └── CODEMAP.md
    └── tests/
        └── CODEMAP.md
    ```
    
    ## 安装
    
    将 `SKILL.md` 复制到 Claude Code 的 skills 目录：
    
    ```bash
    mkdir -p ~/.claude/skills/codemap
    cp SKILL.md ~/.claude/skills/codemap/SKILL.md
    ```
    
    ## 使用
    
    在 Claude Code 会话中触发（以下任一方式）：
    
    - `/codemap`
    - 对 Claude 说 "帮我索引这个项目" / "map this codebase" / "generate codemap"
    
    Skill 会依次询问：
    1. 项目模式（学习 / 维护）
    2. 是否启用并行 sub-agent（默认上限 3）
    3. 额外忽略规则
    
    然后自动扫描、生成所有 CODEMAP.md 和分析文件。
    
    ## 导航工作流
    
    Agent 使用 CODEMAP 的**约束式读取流程**：
    
    ```
    Step 1: 读取根 CODEMAP → Task Guide 匹配当前任务类型
            ↓ 命中 → Target + Also Check 直接给出初始文件集
            ↓ 未命中 → Domain 过滤定位目标目录
    
    Step 2: 并行读取目标子目录 CODEMAP
            ↓ Task Guide 精确定位文件
            ↓ Domain 列排除非匹配文件
            ↓ File Dependencies 仅条件性触发（接口变更/契约理解）
    
    Step 3: 处理跨目录依赖
            ↓ Also Check → 直接读取（经验验证）
            ↓ Exposed To ≤5 → 仅在签名/语义变更时读
            ↓ Exposed To >5 (foundational) → grep → Domain 过滤 → 读
    
    Step 4: 编译最终文件集 → 去重
    
    Step 5: 大文件先读 .analysis.md Feature Index → 定位行范围
    
    Step 6: 并行批量读取所有目标源文件（使用 offset/limit 精准读取大文件行范围）
    
    Step 7 (Stage 2): 仅在分析证明确实需要时补充读取更多文件
    ```
    
    **核心效率原则**：阅读 3-4 个 CODEMAP（~200 行）即可精确定位 5-8 个源文件。Task Guide + Domain 过滤在读取任何源码之前缩小候选集。Feature Index 对大文件实现行级精准读取。依赖关系信息是**安全网而非读取指令**——仅在特定条件触发下才引发额外读取。
    
    ## 更新规则（维护模式）
    
    Agent 在修改代码后，通过以下**决策树**自主判断是否需要更新 CODEMAP：
    
    1. **结构变更**（文件/目录增删移）→ 重新生成受影响目录的全部 CODEMAP 内容
    2. **接口变更**（公开符号签名/名称变更）→ 更新 Key Exports、Cross-Dir Dependencies（含阈值穿越处理），向上传播
    3. **实现变更**（bug fix / 内部重构 / 参数调整）→ 无需更新 CODEMAP（仅在大文件行偏移 >20 行时更新 Logical Sections 行号）
    
    ## 许可证
    
    MIT
    ---
    name: codemap
    description: Use when indexing a codebase for agent navigation, generating or refreshing CODEMAP.md files, mapping large project structure, or adding CODEMAP guidance for learning/maintenance workflows.
    ---
    
    # CODEMAP — Codebase Navigation Index Generator
    
    Generate hierarchical `CODEMAP.md` files that help agents locate relevant code without scanning unrelated files. Core navigation: **Task Guide first → Domain filter → targeted reads**. For files over 1000 lines, generate a companion `<filename>.analysis.md` with intent-to-line-range mapping.
    
    ## Core Principles
    
    - `CODEMAP.md` is a navigation constraint, not documentation to browse.
    - Prefer positive guidance (Task Guide, Domain, Key Exports) over broad listings.
    - Dependencies are a safety net, not an invitation to chain-read.
    - Each CODEMAP describes only its own directory level. Child directory details belong in child CODEMAPs.
    - Maintain a bijection between in-scope source entities and canonical inventory entries: no omissions, no duplicate ownership, no entries for nonexistent entities. Navigation references may be many-to-one; they are not additional inventory entries.
    - Every code change requires considering addition, deletion, and modification together. Maintenance also actively retires superseded code within the authorized task scope; adding an index entry does not complete a migration.
    
    ## Bijection and Freshness
    
    Define the inventory scope using the declared ignore rules. Every included immediate source file has exactly one canonical row in its directory's Files table; every included immediate child directory has exactly one Subdirectories row. Parent maps route to children without duplicating their inventories. CODEMAPs and companion analysis files are index artifacts, not source entities to recursively index. Key Exports and Task Guide are selective navigation views, not exhaustive inventories of all symbols or tasks.
    
    Validate both directions against the current source tree, including uncommitted changes: source → map catches omissions and duplicate ownership; map → source catches stale paths, symbols, responsibilities, dependencies, and line ranges. Do not exclude a source merely to hide an omission. Shared implementations can have multiple callers and navigation references without duplicate canonical entries.
    
    Freshness is a completion condition after each code change, not a timestamp update. In maintenance mode, inspect affected implementations for replacement, duplication, obsolete requirements, and completed migration scaffolding. Retire confirmed obsolete code and its exclusive tests, dependencies, build routes, documentation, and map entries in the same task. Retained compatibility paths need concrete consumers or a current contract and an explicit removal condition; recheck that condition when affected. Age or absence of local text references alone is not proof of obsolescence. Learning mode remains read-only for source code.
    
    ## Language Rule
    
    Generated files use the user's request language for prose/headings. Code identifiers, file names, paths, and symbols keep original spelling.
    
    ## Before Generation: Ask Three Questions
    
    Unless already specified:
    
    1. **Mode**: `Learning` (read-only study) or `Maintenance` (active development).
    2. **Sub-agents**: `Yes, max 3` (recommended), custom limit, or `No`.
    3. **Ignore rules**: `Defaults + .gitignore` (recommended), or add custom patterns.
    
    Mode differences:
    
    | Aspect | Learning | Maintenance |
    |---|---|---|
    | Frontmatter | `mode: learning` | `mode: maintenance`, `commit: <hash>` |
    | Task Guide | suggested entry point | primary navigation, strict |
    | Domain | soft focus hint | hard filter unless justified |
    | Dependencies | reference material | gated by interface/impact rules |
    | Updates | one-time | incremental after code changes |
    
    ## Ignore Rules
    
    Merge in order:
    1. Built-ins: `.git/`, dependency dirs, virtualenvs, build outputs, caches, logs, lockfiles, minified files, binaries, image/font assets, IDE folders.
    2. Project `.gitignore`.
    3. User custom patterns.
    
    Include generated code only if it affects navigation; mark `Generated, do not edit manually`.
    
    Bijection applies only within the declared inventory scope. Low-priority areas may be deliberately excluded or represented only by a directory boundary, according to the user's priorities and project conventions. Record these choices visibly in the root CODEMAP's **Scope and Exclusions** section; never silently omit them.
    
    Use `Path / pattern | Treatment | Reason / basis` rows. Treatment is either `Excluded` (no inventory obligation inside this scope) or `Boundary only` (retain one directory entry marked `Internals not indexed`, without a child map or internal file inventory). Patterns are root-relative; list any included exceptions explicitly. Record default exclusions and `.gitignore` as identifiable rule sources, grouping routine patterns rather than listing every ignored file. Project-specific omissions need concrete paths/patterns and reasons, not merely “defaults”. This section is authoritative; frontmatter `ignore` summarizes or points to it.
    
    Ignoring indexing does not mean code is obsolete, safe to delete, or exempt from correctness checks. If a task touches an excluded area or depends on its contract, inspect what is needed and reconsider the recorded scope. Update the scope decision if it changes; do not claim unindexed internals were verified.
    
    ## Generation Workflow
    
    ### 1. Build Global Context
    
    Read lightweight project context only:
    - Prefer root `README.md` / `README.rst` / `README.txt`.
    - Else metadata: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `pom.xml`, etc.
    - Else infer from structure; mark guesses `inferred, verify against code`.
    
    Produce: purpose, architecture shape, major domains. Omit badges/changelogs.
    
    ### 2. Scan and Measure
    
    After applying ignore rules:
    - Build filtered directory topology.
    - Count source files, lines, size per first-level subdirectory and project total.
    - Files over 1000 lines:
      - `<=5` → generate all `.analysis.md`.
      - `>5` → ask: all, top 5, selected, or none.
    
    ### 3. Dispatch Work
    
    If sub-agents enabled, choose count `K`:
    
    | Project size | K |
    |---|---|
    | `<=3000` lines or `<=500KB` | 1 |
    | `3001-15000` lines or `500KB-3MB` | `min(N, 2)` |
    | `>15000` lines or `>3MB` | `N` |
    
    If line count and size disagree, use the larger `K`. Assign first-level directories by greedy bin packing. Keep root loose files with main agent if small (`<=200` lines), else assign to lightest bin.
    
    Sub-agent prompts: self-contained, plain English. Include compressed global context, output language, ignore rules, assigned directories, large files list, required formats, forbidden paths. No overlapping write ownership.
    
    ### 4. Generate Per-Directory Maps
    
    One `CODEMAP.md` per fully indexed source directory (root + included subdirectories). Excluded and boundary-only interiors do not require maps. Each map describes only the current directory level.
    
    ### 5. Assemble Root and Install Protocol
    
    Read first-level CODEMAP summaries and Task Guides. Write root `CODEMAP.md`, then install the Navigation Protocol block into `AGENTS.md` (preferred) or `CLAUDE.md`. Replace existing block if present; do not append duplicates. If neither file exists, create `AGENTS.md`.
    
    ---
    
    ## CODEMAP.md Structure
    
    ### Frontmatter
    
    ```yaml
    ---
    mode: learning | maintenance
    commit: abc1234f        # maintenance only
    ✓ • 0ms

## Activity

    $ rg -n 'codemap|project-maintenance|references/automation|CODEMAP Navigation Protocol|Before Generation|20 lines' ../README.md ../codemap ../project-maintenance /home/fuurin/.codex/skills/.system/skill-creator/scripts/quick_validate.py; sed -n '155,310p' scripts/maintenance_state.py
    ../README.md:7:- [`codemap/`](codemap/)：生成和维护分层 `CODEMAP.md`，按任务路由代码阅读范围。
    ../README.md:8:- [`project-maintenance/`](project-maintenance/)：在功能开发、重构和后端迁移后同步代码地图、实现、测试、构建入口和文档，并处理已退役内容。
    ../README.md:17:ln -s "$PWD/codemap" ~/.codex/skills/codemap
    ../README.md:18:ln -s "$PWD/project-maintenance" ~/.codex/skills/project-maintenance
    ../README.md:27:`project-maintenance/scripts/maintenance_state.py` 可以记录审查范围并检测后续的代码、测试、配置、文档和代码地图变化：
    ../README.md:30:python3 project-maintenance/scripts/maintenance_state.py inspect --root <project-root>
    ../README.md:31:python3 project-maintenance/scripts/maintenance_state.py record --root <project-root> --reviewed
    ../README.md:32:python3 project-maintenance/scripts/maintenance_state.py check --root <project-root>
    ../README.md:39:- `codemap/` 保留上游 MIT 许可证文件。
    ../project-maintenance/scripts/maintenance_state.py:2:"""Track changes since a deliberate project-maintenance review; never delete code."""
    ../project-maintenance/scripts/maintenance_state.py:18:STATE_PATH = Path(".project-maintenance/state.json")
    ../project-maintenance/scripts/maintenance_state.py:123:        target = maps if path.name in {"CODEMAP.md", "codemap.md"} or (
    ../project-maintenance/references/automation.md:1:# 自动化与 codemap 协作
    ../project-maintenance/references/automation.md:3:需要选择工具、批量核对或动态生成辅助脚本时读本参考。工具提取可重复的事实，agent 判断当前契约与变更影响；读取范围随证据扩展。本文描述两种技能的协作，地图格式与导航规则由可用的 `$codemap` 或项目现有地图维护。
    ../project-maintenance/references/automation.md:7:| 能力 | 复用入口 | project-maintenance 的工作 |
    ../project-maintenance/references/automation.md:9:| 任务路由、Domain 过滤、条件读取消费者 | codemap 的 Task Guide、Files、Dependencies | 用变更证据选择入口，判断公开契约是否受影响 |
    ../project-maintenance/references/automation.md:10:| 大文件功能到行范围的定位 | codemap 的 `.analysis.md` Feature Index | 核对受影响范围仍对应当前源码 |
    ../project-maintenance/references/automation.md:11:| 文件/目录唯一归属、范围和排除项 | codemap 的 Files、Subdirectories、根 Scope and Exclusions | 检查当前工作树的缺漏、重复、幽灵路径并同步 |
    ../project-maintenance/references/automation.md:12:| 地图增删改与父子路由更新 | codemap 维护模式 | 将地图更新纳入实现、文档、构建和验证的完成条件 |
    ../project-maintenance/references/automation.md:16:维护状态的扫描范围与地图的索引范围分别保存在审查记录和根地图中；扫描会包含许多无需索引的配置、fixture 和文档。临时候选清单用于本次操作，不成为另一份权威地图。采用 codemap 时使用其维护模式；普通维护沿用既有配置，只有新建或改变索引范围时才处理相应配置问题。
    ../project-maintenance/references/automation.md:27:| 地图表格与当前目录的批量集合核对 | 先复用 codemap/项目工具；缺少时生成限定格式的临时比较脚本 | 缺漏、重复归属、幽灵路径、越界行范围 |
    ../project-maintenance/references/automation.md:37:2. **通过地图定向**：沿 codemap 的任务路由进入受影响目录，用 Feature Index 确定大文件行范围。有具体契约缺口再查消费者，消费者还可能位于本次未修改的目录。
    ../project-maintenance/references/automation.md:40:5. **闭合影响**：同步 codemap、相关说明、构建路线和有效验证。完成实际审查后才 record，再 check；分页参数只压缩报告，不缩小记录范围。
    ../project-maintenance/references/automation.md:46:当既有命令的组合已经够用时直接组合。有明确的重复计算或格式转换需求时，再写一个窄小辅助脚本。例如 codemap 格式的 Files/Subdirectories 双向集合比较应复用或补到 codemap 的工具归属；维护状态继续使用本技能的单一脚本入口。
    ../project-maintenance/SKILL.md:2:name: project-maintenance
    ../project-maintenance/SKILL.md:27:用工具完成枚举、哈希、差异、引用搜索和结构化提取；agent 根据当前契约判断职责、影响、增删改及验证需求。已有 CODEMAP 时沿用其 `Task Guide → Domain → Target` 和大文件 Feature Index，按证据扩展消费者读取范围。已有格式的日常维护只读取命中的地图部分；首次建图或需要格式规则时再加载 `$codemap`。地图缺失或失真时，用源码和现有工具核对后修补对应条目。
    ../project-maintenance/SKILL.md:29:先取计数、路径和行范围，再读必要内容；完整扫描结果可以留在工具侧。按任务需要选择 `rg`、AST、语言服务器或动态脚本，复用工具及维护状态入口。需要选型、批量核对或自建脚本时，读取[自动化与 codemap 协作](references/automation.md)。
    ../project-maintenance/SKILL.md:41:双射的对象是“明确纳入索引的源码实体 ↔ 唯一权威清单条目”，每个实体恰好归属一条；重导出文件也需纳入。任务入口、符号表和依赖边允许多处指向同一实体。采用 `$codemap` 时沿用其本层 Files/Subdirectories、父子路由及索引产物排除规则；其他地图保留等价归属，使用已有格式。
    ../project-maintenance/SKILL.md:45:双射只约束声明的索引范围。根地图统一记录排除项、仅保留边界的目录、原因及纳入例外；内部未索引的目录只保留唯一边界条目。采用 `$codemap` 时沿用其 Scope and Exclusions 格式，其他位置引用根地图的范围。
    ../project-maintenance/SKILL.md:49:使用 `$codemap` 的维护模式或项目已有的地图格式，更新受影响范围：
    ../project-maintenance/SKILL.md:94:记录位于 `.project-maintenance/state.json`，包含当前文件哈希与审查版本。`check` 在文件或审查范围变化后返回非零，供已有验证命令、pre-commit 或 CI 调用；新增这些集成时使用项目现有方式，编辑一个可重复更新的入口。脚本的范围、排除模式和能力限制见验证参考。
    ../project-maintenance/agents/openai.yaml:4:  default_prompt: "Use $project-maintenance to finish this change by updating code maps, retiring obsolete code and tests, and verifying the current product."
    ../project-maintenance/references/verification.md:44:- tracked fixture、lockfile、配置与文档也在范围内。`CODEMAP.md`、`codemap.md` 和 `*.analysis.md` 在报告中单列；记录文件自身不进入指纹。
    ../project-maintenance/references/verification.md:49:- `record` 只原子替换 `.project-maintenance/state.json`。若使用 CI，可把这份记录交给 Git；后续在同一路径更新，不生成每次任务的副本。
    ../codemap/README.md:1:# codemap-skill
    ../codemap/README.md:64:mkdir -p ~/.claude/skills/codemap
    ../codemap/README.md:65:cp SKILL.md ~/.claude/skills/codemap/SKILL.md
    ../codemap/README.md:72:- `/codemap`
    ../codemap/README.md:73:- 对 Claude 说 "帮我索引这个项目" / "map this codebase" / "generate codemap"
    ../codemap/README_EN.md:1:# codemap-skill
    ../codemap/README_EN.md:64:mkdir -p ~/.claude/skills/codemap
    ../codemap/README_EN.md:65:cp SKILL.md ~/.claude/skills/codemap/SKILL.md
    ../codemap/README_EN.md:72:- `/codemap`
    ../codemap/README_EN.md:73:- Or say: "index this project" / "map this codebase" / "generate codemap"
    ../codemap/README_EN.md:118:3. **Implementation change** (bug fix / internal refactor / param tweak) → No CODEMAP update (only adjust Logical Sections line ranges if offset >20 lines in large files)
    ../codemap/SKILL.md:2:name: codemap
    ../codemap/SKILL.md:31:## Before Generation: Ask Three Questions
    ../codemap/SKILL.md:269:## CODEMAP Navigation Protocol
    ../codemap/SKILL.md:288:## CODEMAP Navigation Protocol
            if temporary is not None and temporary.exists():
                temporary.unlink()
    
    
    def paginate_report(report: dict, max_paths: int, offset: int) -> dict:
        # Keep full counts and review status; pagination is presentation, not scope.
        entries = [
            (group, kind, name)
            for group in ("maps", "files")
            for kind, names in report["changes"][group].items()
            for name in names
        ]
        entries.extend(("separate_checkouts", "", item)
                       for item in report["separate_checkouts"])
        page = entries[offset:offset + max_paths]
        limited = {
            **report,
            "change_counts": {
                group: {kind: len(names) for kind, names in changes.items()}
                for group, changes in report["changes"].items()
            },
            "separate_checkout_count": len(report["separate_checkouts"]),
            "changes": {
                group: {kind: [] for kind in changes}
                for group, changes in report["changes"].items()
            },
            "separate_checkouts": [],
            "pagination": {
                "offset": offset, "limit": max_paths, "total": len(entries),
                "returned": len(page), "omitted": len(entries) - len(page),
                "truncated": len(page) < len(entries),
                "next_offset": offset + len(page) if page and (
                    offset + len(page) < len(entries)
                ) else None,
            },
        }
        for group, kind, item in page:
            if group == "separate_checkouts":
                limited[group].append(item)
            else:
                limited["changes"][group][kind].append(item)
        return limited
    
    
    def print_report(report: dict, as_json: bool) -> None:
        if as_json:
            print(json.dumps(report, indent=2, sort_keys=True))
            return
        print(f"Project: {report['root']}")
        print(f"Status: {report['status']}")
        for group in ("files", "maps"):
            print(f"{group}: {report['counts'][group]} monitored")
            for kind, paths in report["changes"][group].items():
                if paths:
                    print(f"  {kind}: {len(paths)}")
                    for name in paths[:40]:
                        print("    " + ascii(name))
                    if len(paths) > 40:
                        print("    ... use --json for the complete list")
        if report["scope_changed"]:
            print("Scope changed; review exclusions before recording")
        if report["separate_checkouts"]:
            print("Linked/submodule content requires separate review:")
            for checkout in report["separate_checkouts"]:
                print("  " + ascii(checkout["path"]) + " (" + checkout["kind"] + ")")
    
    
    def main() -> int:
        parser = argparse.ArgumentParser(description=__doc__)
        parser.add_argument("action", choices=("inspect", "record", "check"))
        parser.add_argument("--root", required=True, help="Path inside the intended Git project")
        parser.add_argument("--json", action="store_true", help="Emit structured output; complete unless --max-paths is set")
        parser.add_argument("--max-paths", type=int, help="Limit total JSON change/checkout entries; 0 emits counts only")
        parser.add_argument("--offset", type=int, default=0, help="Skip entries when using --max-paths")
        parser.add_argument("--reviewed", action="store_true", help="Record only after actual review")
        parser.add_argument("--exclude", action="append", help="Replace custom root-relative exclusions")
        parser.add_argument("--clear-excludes", action="store_true")
        parser.add_argument("--include-generated", action="store_true", default=None)
        parser.add_argument("--exclude-generated", dest="include_generated", action="store_false", default=None)
        args = parser.parse_args()
        if args.action == "record" and not args.reviewed:
            parser.error("record requires --reviewed after maintenance and verification")
        if args.reviewed and args.action != "record":
            parser.error("--reviewed applies only to record")
        if args.exclude is not None and args.clear_excludes:
            parser.error("Use --exclude or --clear-excludes, not both")
        if args.max_paths is not None and (not args.json or args.max_paths < 0):
            parser.error("--max-paths requires --json and a nonnegative limit")
        if args.offset < 0 or (args.offset and args.max_paths is None):
            parser.error("--offset requires --max-paths and a nonnegative offset")
        try:
            root = repo_root(args.root)
            old = load_state(root)
            patterns = sorted(set(args.exclude)) if args.exclude is not None else (
                [] if args.clear_excludes else (old or {}).get("excludes", [])
            )
            if any(not p or p.startswith("/") or ".." in Path(p).parts for p in patterns):
                raise ValueError("Exclusions must be nonempty root-relative patterns without '..'")
            include_generated = args.include_generated if args.include_generated is not None else (
                (old or {}).get("include_generated", False)
            )
            current = inventory(root, patterns, include_generated)
            changes = {group: differences((old or {}).get(group, {}), current[group])
                       for group in ("files", "maps")}
            scope_changed = old is not None and (
                patterns != old["excludes"] or include_generated != old["include_generated"]
            )
            needs_review = old is None or scope_changed or any(
                paths for group in changes.values() for paths in group.values()
            )
            status = "review-required" if needs_review else "matches-reviewed-state"
            if args.action == "record":
                state = {
                    "schema_version": SCHEMA_VERSION,
                    "reviewed_at": datetime.now(timezone.utc).isoformat(),
                    "head": git(root, "rev-parse", "--verify", "HEAD", optional=True).decode().strip() or None,
                    "excludes": patterns, "include_generated": include_generated, **current,
                }
                write_state(root, state)
                status = "recorded-reviewed-state"
            report = {
                "root": str(root), "status": status, "scope_changed": scope_changed,
                "counts": {group: len(current[group]) for group in ("files", "maps")},
                "changes": changes, "separate_checkouts": current["separate_checkouts"],
                "excludes": patterns, "include_generated": include_generated,
            }
            if args.max_paths is not None:
                snapshot = {
                    "current": current, "reviewed": old,
                    "excludes": patterns, "include_generated": include_generated,
                }
                report["report_id"] = hashlib.sha256(
                    json.dumps(snapshot, sort_keys=True).encode()
                ).hexdigest()
                report = paginate_report(report, args.max_paths, args.offset)
            print_report(report, args.json)
            return int(needs_review) if args.action == "check" else 0
        except (OSError, ValueError, UnicodeError) as error:
            if args.json:
                print(json.dumps({"error": str(error)}), file=sys.stderr)
            else:
                print(f"Error: {error}", file=sys.stderr)
            return 2
    
    
    if __name__ == "__main__":
        sys.exit(main())
    ✓ • 0ms

## Activity

    $ python3 - <<'PY'
        from pathlib import Path
        import hashlib
        import json
        import tempfile
        
        repository = Path('/home/fuurin/code/skills')
        staging = Path(tempfile.mkdtemp(prefix='skill-sync-'))
        drafts = {}
        drafts['codemap/SKILL.md'] = '''---
        name: codemap
        description: Use when indexing a codebase for agent navigation, generating or refreshing CODEMAP.md files, mapping large project structure, or adding CODEMAP guidance for learning/maintenance workflows.
        ---
        
        # CODEMAP
        
        Maintain hierarchical navigation maps so agents reach relevant code through **Task Guide → Domain → targeted reads**. Keep one current map per indexed directory and use `<filename>.analysis.md` for files over 1000 lines. Generated prose follows the user's language; identifiers and paths retain their spelling.
        
        ## Mode and responsibility
        
        - **Learning**: source remains read-only; maps guide study.
        - **Maintenance**: preserve existing scope, format and navigation settings; update only affected maps and routes. Infer the mode from the task, asking only when the distinction matters and is unresolved.
        - This skill owns map structure, scope and navigation. For implementation/test/document retirement, product verification and reviewed-state tracking, use available `$project-maintenance` guidance or the project's existing maintenance workflow. Load only guidance needed for the current task.
        
        ## Navigate before reading
        
        1. Start at root Task Guide. A matching **Target** is the initial read set; **Also Check** is conditional on the task, evidence from target code, or public-contract impact.
        2. Without a task match, filter by **Domain** and descend through local maps. Maintenance reads outside the chosen domain need a concrete reason; learning may use other domains as study context.
        3. For large files, read the matching **Feature Index** line ranges; use **Logical Sections** as fallback. Verify ranges against current source.
        4. Read imported code when its contract is needed. Inspect consumers when changing public signatures, return types or semantics. Search and filter large consumer sets before reading; expand dependencies only for a remaining contract gap.
        5. Deduplicate and batch independent targeted reads. A missing or stale route calls for a scoped source query and map repair.
        
        ## Inventory and scope
        
        Every included immediate source file belongs to exactly one local **Files** row, including pure re-exports; every included immediate child directory belongs to exactly one **Subdirectories** row. Each map describes its own level. Task Guide, exports and dependency links may reference the same entity without creating duplicate ownership. Maps and companion analyses are index artifacts, excluded from recursive source inventory.
        
        Root **Scope and Exclusions** is authoritative: record root-relative paths/patterns, treatment (**Excluded** or **Boundary only**), reason and included exceptions. Boundary-only directories retain one marked route; their interiors need no inventory or child maps. Routine defaults and `.gitignore` may be grouped by identifiable rule source; project-specific omissions need explicit paths and reasons.
        
        Use defaults for VCS/dependency/cache/build directories, logs, lockfiles, minified/binary/assets and IDE files, then apply project and user rules. Include generated code when navigation requires it and mark it generated. Reconsider scope when work touches excluded code or depends on its contract; exclusion supplies no deletion or correctness evidence. Maintenance scan exclusions remain separate from map scope.
        
        Check the working tree both ways: source → map for missing/duplicate ownership; map → source for ghost paths and stale responsibilities, symbols, dependencies or ranges. Scope changes require a current rationale; widening exclusions to conceal omissions does not complete maintenance.
        
        ## Tools and agent judgment
        
        Use Git and `rg` for changes, paths and text candidates. For structural queries or uniform rewrites, use available **ast-grep**; for symbol identity/references, use a configured language server. Check local capabilities before choosing a command. Tools extract facts; the agent decides domains, intent, responsibilities and affected contracts.
        
        Return counts, paths, symbols and necessary ranges before full content. Keep large raw results on the tool side and disclose truncation. Reuse project helpers; a temporary script can normalize a known map format or compare scoped sets. Unknown formats and read/parse errors must remain visible. Text/AST matches alone cannot prove retirement or a complete call graph.
        
        ## Create or update maps
        
        - New maps: read lightweight project context (README or package metadata), collect filtered topology and source sizes with tools, then inspect only code needed to establish responsibilities. Mark unverified inferences.
        - Read [map and analysis format](references/format.md) when generating maps or changing their schema. Existing-format maintenance can use local tables directly.
        - Generate maps for fully indexed directories; parent maps route to children without copying child inventories. Monorepos may use package roots and a top-level package index. If many large files need analyses, agree on their coverage when task scope does not resolve it.
        - Add/delete/move/rename: update local canonical entries, affected parent routes and Task Guides; remove index artifacts for retired sources/directories. Interface, responsibility or dependency changes also update exports, consumers and large-file ranges.
        - Evaluate **addition, deletion and modification** within the authorized maintenance task. Recheck replacement and consumer migration before retiring old paths and exclusive resources. Retained compatibility paths need current consumers/contracts and a checkable removal condition; age or no local text references is insufficient.
        - Pure internal equivalent changes may leave maps intact after checking indexed meaning and ranges. Completion reports applicable operations, evidence and remaining gaps; timestamps and successful builds alone do not establish freshness.
        - First generation installs one short navigation block following [project protocol](references/protocol.md); later edits update it only when navigation rules change. Preserve other project instructions.
        
        Parallel agents are optional when authorized: assign independent directory ownership and pass only needed context. Use the task's existing configuration; choose delegation when its coordination cost is justified.
        '''
        drafts['codemap/references/format.md'] = '''# Map and analysis format
        
        Read when generating maps or changing their schema. Section names below describe the roles; translate prose/headings into the user's language while keeping paths and identifiers unchanged.
        
        ## CODEMAP.md
        
        ```yaml
        ---
        mode: maintenance
        commit: abc1234f
        ignore: See Scope and Exclusions
         generated_at: YYYY-MM-DD
        stats:
          total_files: 114
          total_lines: 18200
          total_size: 4.2 MB
        ---
        ```
        
        Use `mode: learning` for study maps and omit `commit`. `commit` records the base revision, not proof of a clean worktree. `ignore` and `stats` belong to root only; date and measurements describe the actual generation.
        
        Use this section order, omitting empty sections except the root scope record:
        
        | Section | Placement | Columns / content |
        | --- | --- | --- |
        | Summary | Every map | One sentence about this directory |
        | Scope and Exclusions | Root | Path / pattern · Treatment · Reason / basis |
        | Task Guide | Every functional map | Task · Domain · Target · Also Check |
        | Subdirectories | Maps with included children | Dir · Domain · Depends On · Purpose |
        | Key Exports | Where cross-directory users need entry points | Symbol · Source · Line |
        | Files | Maps with included immediate files | File · Domain · Deps (if useful) · Function |
        | File Dependencies | Where same-directory relationships help | File · Imports (in-dir) · Exposed To (in-dir) |
        
        A container with no immediate source files needs Summary, Task Guide and Subdirectories; root also records scope. Excluded and boundary-only interiors need no maps.
        
        ### Scope and Exclusions
        
        Patterns are root-relative. Treatment is **Excluded** or **Boundary only**. For the latter, mark the retained Subdirectories entry **Internals not indexed**. Group routine defaults and `.gitignore` by their rule sources; name project-specific omissions, reasons and included exceptions. Root is the single authoritative scope record.
        
        ### Task Guide and domains
        
        Use concrete study/modification scenarios. **Target** names the primary read set; **Also Check** lists short conditional candidates, using the navigation gates in SKILL.md. Cover functional subdirectories and common tasks; missing scenarios fall back to Domain and Key Exports.
        
        Domains identify meaningful functional areas and agree with local Files/Subdirectories values. Mark trivial re-exports with `—` where no domain helps. A large flat directory may group Files rows by domain.
        
        ### Canonical inventory and exports
        
        Files/Subdirectories hold one row per included immediate entity. Preserve pure re-export files. Describe responsibilities in one concise sentence; large-file rows link their companion analysis. Parent inventories contain child routes rather than child files or symbols.
        
        Key Exports is a selective view of immediate-source symbols used from other directories. Use `L:<number>` locations and keep the important entry points, typically about 15 or fewer. Navigation references do not replace canonical inventory rows.
        
        ### Dependencies
        
        Subdirectories uses directory-level internal/external dependencies. File Dependencies contains only relationships between immediate files in the same directory. Cross-directory relationships belong to Files **Deps**:
        
        - `← path`: a file this source imports.
        - `→ path`: a consumer of this source.
        - More than five consumers: use a count and scoped search command, e.g. `→ N files (foundational); rg -l "SymbolName" src`.
        - Omit the Deps column when it adds no cross-directory information; use `—` for empty cells otherwise. Escape literal pipes inside Markdown cells.
        
        Consumers may span domains; inspect them when public contract changes justify it. Lists and search commands must match current source and configuration.
        
        ## Large-file companion
        
        Place `<filename>.analysis.md` beside source files over 1000 lines within the agreed analysis coverage:
        
        ```markdown
        ---
        source: filename.py
        lines: 1842
        generated_at: YYYY-MM-DD
        ---
        
        > One-sentence summary.
        
        ## Feature Index
        
        | Intent | Lines | Notes |
        | --- | --- | --- |
        
        ## Symbols
        
        | Symbol | Type | Line |
        | --- | --- | --- |
        
        ## Logical Sections
        
        | Lines | Content |
        | --- | --- |
        ```
        
        Feature Index maps concrete intents to current line ranges; Notes describe same-file coupling. Symbols lists useful top-level public classes, functions and constants. Logical Sections provides about 5–10 broad segments as fallback. Add Class Hierarchy only when depth greater than two helps navigation. Keep analysis focused on location and intent; omit code snippets and duplicated implementation/API descriptions.
        '''.replace('\n generated_at:', '\ngenerated_at:')
        drafts['codemap/references/protocol.md'] = '''# Project navigation protocol
        
        Read when first installing the protocol or changing navigation rules. Use `AGENTS.md`, or the project's existing `CLAUDE.md`; create `AGENTS.md` when neither exists. Replace the existing CODEMAP protocol section in place and preserve surrounding instructions. Keep one block, with the selected mode and links to the actual root map.
        
        The following compact block is a template. Set Mode to the selected mode; adjust root paths for package-level maps.
        
        ```markdown
        ## CODEMAP Navigation Protocol
        
        Mode: maintenance. Start navigation at `CODEMAP.md`.
        
        - Match Task Guide first; Target is the initial read set. Also Check requires task relevance, evidence from target code, or public-contract impact.
        - Without a match, descend through matching Domain routes and local Task Guides. Maintenance expansion needs a concrete reason.
        - For large files, use companion `.analysis.md` Feature Index ranges; Logical Sections is the fallback.
        - Read imports for a needed interface contract, consumers for public signature/return/semantics changes. Search and filter large result sets before reading; extend dependencies for unresolved contract gaps.
        - Batch independent targeted reads. Verify stale paths/ranges against current source.
        - Maintenance evaluates addition, deletion and modification: update affected canonical entries, parent routes, exports and ranges. Check current source ↔ map within root Scope and Exclusions; each included file/directory has one local owner. Excluded/boundary-only interiors follow the declared scope.
        - Retire confirmed superseded paths within the authorized task after checking real consumers and replacement; retain compatibility only with a current contract and removal condition. Use the project maintenance workflow for product verification and linked resources.
        - Learning keeps source read-only. Report incomplete scope or verification explicitly.
        ```
        
        This block points agents into current maps. Keep schema details in the skill's format reference and current project scope in the root map.
        '''
        drafts['codemap/README.md'] = '''# codemap-skill
        
        **中文** | [English](README_EN.md)
        
        生成和维护层级化 `CODEMAP.md`，让 agent 通过任务和功能域定位代码。超过 1000 行的文件可配套 `<filename>.analysis.md`，按功能定位行范围。
        
        ## 使用
        
        在支持 skills 的 agent 中请求“索引这个项目”或“更新受影响的 CODEMAP”。学习模式保持源码只读；维护模式沿用已有范围与格式，增量更新地图。仅在需求无法确定模式或范围时询问；并行 agent 按授权和实际收益选用。
        
        从本仓库根目录安装完整目录，例如：
        
        ```bash
        mkdir -p ~/.claude/skills
        cp -a codemap ~/.claude/skills/
        ```
        
        安装到 Codex 时可将目的目录换为 `~/.codex/skills/`。保留 `references/`，技能正文会按需引用格式和协议说明。
        
        ## 导航与维护
        
        - 先匹配 Task Guide 的 Target；Also Check 仅在任务、当前代码证据或公开契约影响需要时读取。
        - 未命中任务时按 Domain 逐层进入；大文件先定位 Feature Index 的行范围。
        - 接口契约需要时读 imports，公开签名或语义改变时查 consumers；批量读取前去重、筛选。
        - 根地图声明范围、排除项和仅保留边界的目录。范围内源码与本层权威条目一一对应，包括重导出文件。
        - 变更后评估增删改，核对职责、接口、依赖和行范围；纯内部等价修改须确认索引仍准确。退役依据来自当前需求、替代实现和消费者迁移。
        
        Git、rg 提供差异和文本候选，ast-grep 按需提取结构或预览批量改写，语言服务器查询真实符号关系；agent 判断职责与影响。完整结果留在工具侧，优先返回路径、行号与必要片段。
        
        ## 与 project-maintenance 协作
        
        codemap 负责地图结构、范围和导航；project-maintenance 负责实现、测试、文档和构建路线的退役、产品验证及审查状态。日常维护沿用现有地图，只加载当前任务需要的说明。
        
        - [技能入口](SKILL.md)
        - [地图与大文件格式](references/format.md)
        - [项目导航协议](references/protocol.md)
        
        [MIT License](LICENSE)
        '''
        drafts['codemap/README_EN.md'] = '''# codemap-skill
        
        [中文](README.md) | **English**
        
        Generate and maintain hierarchical `CODEMAP.md` indexes so agents locate code by task and functional domain. Files over 1000 lines can have `<filename>.analysis.md` companions that map intents to line ranges.
        
        ## Usage
        
        Ask a skill-capable agent to “index this project” or “refresh the affected CODEMAPs.” Learning keeps source read-only; maintenance inherits current scope/format and updates maps incrementally. Ask about mode or scope only when unresolved. Parallel agents depend on authorization and actual benefit.
        
        Install the complete folder from the repository root, for example:
        
        ```bash
        mkdir -p ~/.claude/skills
        cp -a codemap ~/.claude/skills/
        ```
        
        For Codex, the destination can be `~/.codex/skills/`. Keep `references/`; the skill loads format and protocol details when needed.
        
        ## Navigation and maintenance
        
        - Match Task Guide Target first. Read Also Check only for task relevance, evidence from current code, or public-contract impact.
        - Otherwise descend by Domain; large files use Feature Index line ranges.
        - Read imports for a needed interface contract, consumers for public signature/semantics changes. Deduplicate and filter before batch reads.
        - Root declares scope, exclusions and boundary-only directories. Included source entities have unique local inventory entries, including re-export files.
        - Evaluate addition, deletion and modification after changes. Check responsibilities, interfaces, dependencies and ranges before leaving a map intact. Retirement follows current requirements, replacement and consumer migration evidence.
        
        Git and rg supply changes/text candidates; ast-grep can extract structure or preview uniform rewrites; language servers resolve symbol relationships. Agents judge responsibilities and impact. Keep raw results on the tool side and return paths, lines and necessary snippets first.
        
        ## Working with project-maintenance
        
        codemap owns map structure, scope and navigation. project-maintenance owns implementation/test/document/build-route retirement, product verification and reviewed-state tracking. Routine maintenance reuses current maps and loads only task-relevant guidance.
        
        - [Skill entry point](SKILL.md)
        - [Map and large-file format](references/format.md)
        - [Project navigation protocol](references/protocol.md)
        
        [MIT License](LICENSE)
        '''
        drafts['project-maintenance/SKILL.md'] = '''---
        name: project-maintenance
        description: Maintain code maps, implementations, tests, build routes and docs after feature work, refactoring or migrations; retire superseded code and track reviewed state, including native UI and inference engines.
        ---
        
        # Project Maintenance
        
        把维护作为当前开发任务的完成条件：同步受影响的地图，处理已替代实现与专属资源，验证当前产品契约。在本次授权范围内完成工作；支持平台、外部接口或任务范围改变时先说明具体影响。
        
        ## 定位范围并取得事实
        
        读取项目规则、当前需求和已有地图，保留语言、平台、引擎及源码所有权约定。结合 Git 已提交、暂存、未暂存和新增文件区分本次任务及其他未完成工作。链接源码、子模块和独立 checkout 在各自真实目录核对。
        
        ```bash
        python3 <skill-dir>/scripts/maintenance_state.py inspect --root <project-root> --json --max-paths 20
        ```
        
        脚本比较上次审查记录与当前内容；首次报告是待建立基线，全部 added 不等于本次新增。分页保留完整计数与状态，按[验证参考](references/verification.md)读取后续项。没有 Git 时按显式目录审查；只有项目创建任务已授权初始化仓库时才建立 Git。
        
        已有 CODEMAP 时从 Task Guide/Domain 定位目标，大文件使用 Feature Index；按契约影响读取消费者。地图格式、范围和父子路由归可用的 `$codemap` 或项目既有规范；日常维护只读取匹配部分，首次建图或调整格式时再加载相应生成说明。
        
        工具提取差异、路径、引用和结构，agent 判断职责、影响与维护操作。先取计数、路径和行范围，再读必要内容。结构搜索/统一改写按需使用 ast-grep，符号关系使用现有语言服务器；批量核对或自建工具时读[自动化参考](references/automation.md)。
        
        ## 增删改一起评估
        
        每次变更评估三类操作，执行适用项；项目更新包含任务范围内的补缺与去冗余，无需另等清理指令。
        
        | 操作 | 完成条件 |
        | --- | --- |
        | 增 | 补上当前契约需要的实现、有效验证和地图条目 |
        | 删 | 清除已确认退役或冗余的代码及专属测试、依赖、构建路线、文档、fixture 和地图条目 |
        | 改 | 原地修正仍承担职责的实现，同步接口、依赖、路径、消费者和大文件行范围 |
        
        以当前工作树双向核对源码与地图：纳入范围的实体恰好归属一个权威清单条目；导航与依赖允许多处引用。根地图统一声明排除项、仅保留边界的目录、理由及纳入例外。触及未索引区域时重新判断范围与契约；维护扫描排除和地图索引排除分别使用，不能用于掩盖缺漏或作为删除依据。
        
        文件增删移动需同步父目录路由，接口/职责变化需同步导出、消费者和数据流。纯内部等价修改须确认地图语义与行范围仍准确才可不改。地图和有效说明各保留当前权威版本，项目规则中只放简短导航入口。
        
        ## 退役实现、测试和文档
        
        - 主动复查受影响旧实现和临时层；替代已验证且消费者已迁移，或旧需求已撤销时，本次完成退役。保留项写明当前消费者/契约及可检查的移除条件，后续相关变更重新核对。
        - 先查调用者、构建目标、平台及公开契约；动态注册、外部导出、FFI、汇编和函数指针需要核对真实消费者。仅有无本地文本引用、文件年龄、失败/跳过或低覆盖率都不足以删除；报告剩余不确定性。
        - 退役同时处理专属测试、fixture、依赖、配置、文档、示例和导航；核对共享资源与反向链接。Git 留存历史，避免累积 `old/`、`v1/`、`backup/` 副本。
        - 合并重复或仅验证 mock 自身的测试，保留真实回归、不同输入、平台和边界断言；失效需求的测试按当前契约退役或重写。新验证针对实际行为/故障，不能靠弱化断言、放宽误差或重录错误快照使检查通过。
        - 更新受影响 README、运行说明、命令、路径、链接及状态描述，合并重复说明；以当前契约与消费者判断文档是否应删除。
        
        ## 验证并记录
        
        按[验证与退役判据](references/verification.md)选择涉及的语言、接口、UI 或推理验证，复用项目入口。有效审查和验证完成后才记录：
        
        ```bash
        python3 <skill-dir>/scripts/maintenance_state.py record --root <project-root> --reviewed
        python3 <skill-dir>/scripts/maintenance_state.py check --root <project-root>
        ```
        
        `--reviewed` 表示 agent 已实际完成审查和必要验证；状态保存在 `.project-maintenance/state.json`。指纹一致只说明之后没有受监测变化，不能替代地图语义或产品正确性。未闭合改动、未知失败及未验证的其他源码树须如实报告，不能为让检查变绿而重录。
        
        最终简述地图同步、增删改及不适用依据、退役/保留理由、实际验证结果和剩余缺口。
        '''
        drafts['project-maintenance/references/automation.md'] = '''# 自动化与 agent 协作
        
        批量核对、选工具或编写辅助脚本时读本参考。地图格式和导航沿用 codemap/项目已有规范；维护状态复用 `maintenance_state.py`。
        
        ## 工具选择
        
        | 需求 | 优先工具 | 给 agent 的结果 |
        | --- | --- | --- |
        | 变更与文件候选 | Git、`rg --files`、维护状态脚本 | 状态、完整计数、路径与差异片段 |
        | 文本消费者 | `rg -l` 后对选定文件用 `rg -n -F` | 去重路径及必要匹配行 |
        | 结构、同类调用与统一改写 | 已安装的 ast-grep，先检查本地 `--help` | 节点/符号、范围及替换预览 |
        | 同名符号区分、引用与语义重命名 | 项目现有语言服务器 | 有效配置与索引覆盖下的定义/引用位置 |
        
        ast-grep 的结构匹配与 JSON 输出适合在工具侧筛选结果；批量写入前检查匹配和替换预览。[官方 CLI](https://ast-grep.github.io/reference/cli.html)。它缺少完整类型和数据流信息，文本/AST 无命中不能证明代码可退役；仍需核对忽略项、动态注册、FFI、导出与平台分支。[能力边界](https://ast-grep.github.io/advanced/tool-comparison)。
        
        ## 控制上下文
        
        先通过地图确定范围，再搜索路径/符号和读取行段。完整结果较大时留在临时文件或工具侧，返回完整计数、必要片段、截断情况及后续入口；不能把不完整结果当成无引用。消费者可能位于本次未修改的目录。
        
        源码变化后重新核对缓存、行号和片段。独立查询可批量执行；只有已授权且可独立分工时才使用多 agent，其总调用成本需要计入收益。分页与扫描范围的具体行为见[验证参考](verification.md)。
        
        ## 动态辅助脚本
        
        既有命令可完成任务时直接组合；重复计算或格式转换需要时再写窄小脚本，放在允许的临时/本地工具目录。明确根目录、输入格式、声明范围和输出位置；使用参数化子进程调用，避免把仓库文本拼成 shell 命令。
        
        地图集合比较只解析权威 Files/Subdirectories；按根地图处理排除、纳入例外和仅保留边界，避免把导航链接算作重复归属。未知格式、解析/读取失败要显式失败。输出完整计数、差异、来源内容标识、必要行号及遗漏情况；批量编辑核对当前内容仍匹配预期。
        
        一次性脚本无需长期保留；真实复用证明有收益后归入对应 skill/项目工具，并补使用说明及必要验证。地图比较归 codemap，维护指纹归本技能，保留单一入口。
        '''
        
        manifest = {}
        for relative, content in drafts.items():
            original = repository / relative
            target = staging / relative
            target.parent.mkdir(parents=True, exist_ok=True)
            target.write_text(content, encoding='utf-8')
            original_bytes = original.read_bytes() if original.exists() else None
            original_text = original_bytes.decode('utf-8') if original_bytes is not None else ''
            manifest[relative] = {
                'original_sha256': hashlib.sha256(original_bytes).hexdigest() if original_bytes is not None else None,
                'old_lines': len(original_text.splitlines()), 'new_lines': len(content.splitlines()),
                'old_chars': len(original_text), 'new_chars': len(content),
            }
        (staging / 'manifest.json').write_text(json.dumps(manifest, indent=2) + '\n')
        print(json.dumps({'staging': str(staging), 'files': manifest}, ensure_ascii=False, indent=2))
        PY
    {
      "staging": "/tmp/skill-sync-rrk9kz_b",
      "files": {
        "codemap/SKILL.md": {
          "original_sha256": "664a850f2a1b81eff7c59f0e62c1613b0963e512a042d7e4ad8bcb5e65eb4ddc",
          "old_lines": 337,
          "new_lines": 50,
          "old_chars": 19100,
          "new_chars": 6261
        },
        "codemap/references/format.md": {
          "original_sha256": null,
          "old_lines": 0,
          "new_lines": 92,
          "old_chars": 0,
          "new_chars": 4541
        },
        "codemap/references/protocol.md": {
          "original_sha256": null,
          "old_lines": 0,
          "new_lines": 22,
          "old_chars": 0,
          "new_chars": 2053
        },
        "codemap/README.md": {
          "original_sha256": "ec6e9d6f8a326a656b8ef3acb5307649089402021abce4da3a476bbb0c56d2d8",
          "old_lines": 122,
          "new_lines": 38,
          "old_chars": 3503,
          "new_chars": 1059
        },
        "codemap/README_EN.md": {
          "original_sha256": "66c826a008f537d58c79df4b02f2efe90dfa83cc93885db9864ce83bf127194a",
          "old_lines": 122,
          "new_lines": 38,
          "old_chars": 6281,
          "new_chars": 2346
        },
        "project-maintenance/SKILL.md": {
          "original_sha256": "f9eaaf8b80931d5a09b5b5b0606c20874568c33e5b854407b6322c623a73a5b3",
          "old_lines": 96,
          "new_lines": 57,
          "old_chars": 4208,
          "new_chars": 2349
        },
        "project-maintenance/references/automation.md": {
          "original_sha256": "8489b5981e64b475408641032ff522819d5f0ee591189320aee5a005cb2de9c5",
          "old_lines": 52,
          "new_lines": 28,
          "old_chars": 3167,
          "new_chars": 1137
        }
      }
    }
    ✓ • 0ms

## Assistant

精简稿已完成：codemap 入口从 337 行缩到 50 行，project-maintenance 从 96 行缩到 57 行。格式细节仍在按需参考中；重复的生成问答、固定 agent 调度表和工具目录已移除。两份技能都保留了索引双向核对、排除项声明、退役证据和验证要求。

## Activity

    file changes: Completed · 1 changes
    Update { move_path: None }: /tmp/skill-sync-rrk9kz_b/project-maintenance/SKILL.md
    @@ -19,3 +19,3 @@
     
    -已有 CODEMAP 时从 Task Guide/Domain 定位目标，大文件使用 Feature Index；按契约影响读取消费者。地图格式、范围和父子路由归可用的 `$codemap` 或项目既有规范；日常维护只读取匹配部分，首次建图或调整格式时再加载相应生成说明。
    +新项目随首个可运行实现建立地图。已有 CODEMAP 时从 Task Guide/Domain 定位目标，大文件使用 Feature Index；按契约影响读取消费者。地图格式、范围和父子路由归可用的 `$codemap` 或项目既有规范；日常维护只读取匹配部分，首次建图或调整格式时再加载相应生成说明。
     

## Activity

    $ python3 - <<'PY'
        from pathlib import Path
        import hashlib
        import json
        
        repository = Path('/home/fuurin/code/skills')
        staging = Path('/tmp/skill-sync-rrk9kz_b')
        manifest_path = staging / 'manifest.json'
        manifest = json.loads(manifest_path.read_text())
        relative = 'project-maintenance/references/verification.md'
        original = repository / relative
        original_bytes = original.read_bytes()
        original_text = original_bytes.decode()
        paragraph = '将核对出的契约写回受影响地图：ABI/FFI 的真实符号、调用约定、布局/偏移、所有权及回调线程；UI 的事件到状态到渲染/推理路径；模型的 shape、dtype、布局、量化和转换边界。\n\n'
        content = original_text.replace('## 语言与接口边界\n', paragraph + '## 语言与接口边界\n', 1)
        (staging / relative).write_text(content)
        manifest[relative] = {
            'original_sha256': hashlib.sha256(original_bytes).hexdigest(),
            'old_lines': len(original_text.splitlines()), 'old_chars': len(original_text),
        }
        for relative, item in manifest.items():
            draft_text = (staging / relative).read_text()
            item['new_lines'] = len(draft_text.splitlines())
            item['new_chars'] = len(draft_text)
        manifest_path.write_text(json.dumps(manifest, indent=2) + '\n')
        print(json.dumps({'files': len(manifest), 'entry_chars_before': sum(manifest[p]['old_chars'] for p in ['codemap/SKILL.md', 'project-maintenance/SKILL.md']), 'entry_chars_after': sum(manifest[p]['new_chars'] for p in ['codemap/SKILL.md', 'project-maintenance/SKILL.md'])}))
        PY
    {"files": 8, "entry_chars_before": 23308, "entry_chars_after": 8626}
    ✓ • 0ms

## Activity

    $ python3 - <<'PY'
        from pathlib import Path
        import hashlib
        import json
        import re
        import yaml
        
        repository = Path('/home/fuurin/code/skills')
        staging = Path('/tmp/skill-sync-rrk9kz_b')
        manifest = json.loads((staging / 'manifest.json').read_text())
        checked_links = 0
        for relative, item in manifest.items():
            source = repository / relative
            digest = hashlib.sha256(source.read_bytes()).hexdigest() if source.exists() else None
            assert digest == item['original_sha256'], f'Concurrent edit: {relative}'
            content = (staging / relative).read_text()
            for target in re.findall(r'\[[^\]]*\]\(([^)]+)\)', content):
                if '://' in target or target.startswith('#'):
                    continue
                target = target.split('#', 1)[0]
                draft_target = (staging / relative).parent / target
                actual_target = source.parent / target
                assert draft_target.exists() or actual_target.exists(), (relative, target)
                checked_links += 1
            for block in re.findall(r'```yaml\n(.*?)\n```', content, re.S):
                if block.startswith('---\n'):
                    block = block.removeprefix('---\n').removesuffix('\n---')
                assert isinstance(yaml.safe_load(block), dict), relative
        print(json.dumps({'originals_unchanged': True, 'local_links_valid': checked_links, 'yaml_templates_valid': True}))
        PY
    {"originals_unchanged": true, "local_links_valid": 16, "yaml_templates_valid": true}
    ✓ • 0ms

## Activity

    $ python3 /home/fuurin/.codex/skills/.system/skill-creator/scripts/quick_validate.py /tmp/skill-sync-rrk9kz_b/project-maintenance
    Skill is valid!
    ✓ • 0ms

## Activity

    $ python3 /home/fuurin/.codex/skills/.system/skill-creator/scripts/quick_validate.py /tmp/skill-sync-rrk9kz_b/codemap
    Skill is valid!
    ✓ • 0ms

## Activity

    $ cp -a /tmp/skill-sync-rrk9kz_b/project-maintenance/. /home/fuurin/code/skills/project-maintenance/
    ✓ • 0ms
