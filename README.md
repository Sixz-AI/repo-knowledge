# RepoKnowledge

> 为 AI 编程助手构建的渐进式代码库知识图谱。  
> A progressive codebase knowledge base plugin for AI coding assistants.

克隆任意 Git 仓库，自动为每个函数/类生成文档，构建可搜索的知识缓存，每次查询后自动变得更聪明。

Clone any Git repo, auto-generate per-function/class documentation, and build a searchable cache that grows smarter with every query.

## 核心价值 / Value

- **无需嵌入向量** — 基于 Agent 语义匹配，不依赖向量数据库
- **渐进式学习** — 每次成功搜索自动添加别名，下次更快找到
- **增量更新** — 只重建变更文件的文档，不重跑全量索引
- **跨语言** — 支持 TS/JS/Python/Go/Java/Rust/C/C++ 等主流语言

## 安装 / Install

```bash
# 克隆仓库 / Clone the repo
git clone https://github.com/code60-AI/repo-knowledge.git
cd repo-knowledge

# 以插件模式启动 Claude Code / Launch Claude Code with plugin
claude --plugin-dir .
```

## 快速命令 / Quick Reference

所有命令使用 `repo-knowledge:` 命名空间前缀，由 Claude Code 插件系统自动识别。

All commands use the `repo-knowledge:` namespace prefix, auto-registered by Claude Code's plugin system.

| 命令 | 说明 | Command |
|------|------|---------|
| `/repo-knowledge:rk-create <git-url>` | 索引新仓库 | Index a new repo |
| `/repo-knowledge:rk-search <query>` | 语义搜索 | Semantic search |
| `/repo-knowledge:rk-update <project>` | 增量更新 | Incremental update |
| `/repo-knowledge:rk-list` | 列出所有知识库 | List all knowledge bases |
| `/repo-knowledge:rk-delete <project>` | 删除知识库 | Delete a knowledge base |

## 文档 / Documentation

- [Getting Started](docs/getting-started.md) — 快速上手指南 ← **从这里开始 / Start here**
- [Epic & Features](docs/epic.md) — 产品目标与功能层次
- [Use Cases](docs/user-stories.md) — 用户故事与使用场景
- [Architecture](docs/architecture.md) — 架构与工作原理
- [Skill Reference](docs/skill-reference.md) — Skill API 参考

## 数据目录 / Data Directory

```
~/.repo-knowledge/
├── _registry.md          # 已注册项目列表
├── _repos/               # 克隆的 Git 仓库
└── <project>/            # 知识缓存
    ├── _meta.md          # 项目元数据
    ├── _index.md         # 带别名的可搜索索引
    └── docs/             # 缓存的文档文件
```

## License

MIT
