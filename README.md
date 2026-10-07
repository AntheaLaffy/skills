# Skills

面向 Codex、Claude Code 和其他编程智能体的本地 skill 集合。每个子目录都是一个可以独立加载的 skill，使用时直接读取对应目录中的 `SKILL.md`。

## 内容

- [`codemap/`](codemap/)：生成和维护分层 `CODEMAP.md`，按任务路由代码阅读范围。
- [`project-maintenance/`](project-maintenance/)：在功能开发、重构和后端迁移后同步代码地图、实现、测试、构建入口和文档，并处理已退役内容。
- [`performance-gradient-optimization/`](performance-gradient-optimization/)：用可复现实验优化延迟、吞吐、CPU、内存、I/O、能耗和成本。
- [`the missing semester/`](the%20missing%20semester/)：The Missing Semester 主题的中文学习型 skills 与练习资料。

## 安装

将需要的 skill 目录链接或复制到工具的 skill 目录。例如：

```bash
ln -s "$PWD/codemap" ~/.codex/skills/codemap
ln -s "$PWD/project-maintenance" ~/.codex/skills/project-maintenance
```

Claude Code 可以使用 `~/.claude/skills/`；如果两个工具共享安装目录，保持同一个源码目录，避免副本漂移。

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
