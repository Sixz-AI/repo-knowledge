# RepoKnowledge

> A progressive codebase knowledge base plugin for AI coding assistants.

Clone any Git repo, auto-generate per-function/class documentation, and build a searchable cache that grows smarter with every query.

## Value

- **No vector embeddings** -- Agent-powered semantic matching, no vector database required
- **Progressive learning** -- Every successful search adds an alias; future queries resolve faster
- **Incremental updates** -- Only rebuilds docs for changed files, never re-indexes the full repo
- **Multi-language** -- Supports TS/JS/Python/Go/Java/Rust/C/C++/Ruby/Swift/Kotlin and more

## Install

```
/plugin marketplace add Sixz-AI/repo-knowledge
/plugin install repo-knowledge@repo-knowledge
```

## Quick Reference

All commands use the `repo-knowledge:` namespace prefix, auto-registered by Claude Code's plugin system.

| Command | Description |
|---------|-------------|
| `/repo-knowledge:rk-create <git-url>` | Index a new repo |
| `/repo-knowledge:rk-search <query>` | Semantic search |
| `/repo-knowledge:rk-update <project>` | Incremental update |
| `/repo-knowledge:rk-list` | List all knowledge bases |
| `/repo-knowledge:rk-delete <project>` | Delete a knowledge base |

## Documentation

> Chinese docs: [docs/zh/](docs/zh/)

- [Getting Started](docs/en/getting-started.md) <- Start here
- [Epic & Features](docs/en/epic.md)
- [Use Cases](docs/en/user-stories.md)
- [Architecture](docs/en/architecture.md)
- [Skill Reference](docs/en/skill-reference.md)

## Data Directory

```
~/.repo-knowledge/
|-- _registry.md          # All registered projects
|-- _repos/               # Cloned git repositories
`-- <project>/            # Knowledge cache
    |-- _meta.md          # Project metadata
    |-- _index.md         # Searchable index with aliases
    `-- docs/             # Cached documentation files
```

## License

MIT
