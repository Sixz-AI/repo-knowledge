# Architecture & Principles

## System Components

```
┌──────────────────────────────────────────────────────────────────────┐
│                              Claude Code                             │
│                                                                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │rk-create │ │rk-search │ │rk-update │ │rk-list   │ │rk-delete │  │
│  │  Skill   │ │  Skill   │ │  Skill   │ │  Skill   │ │  Skill   │  │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘  │
│       │             │             │             │             │        │
│  ┌────▼─────┐ ┌────▼─────┐ ┌────▼─────┐       │             │        │
│  │ Indexer  │ │ Searcher │ │  Update  │       │             │        │
│  │  Agent   │ │  Agent   │ │  Agent   │       │             │        │
│  └────┬─────┘ └────┬─────┘ └────┬─────┘       │             │        │
└───────┼─────────────┼─────────────┼─────────────┼─────────────┼────────┘
        │             │             │             │             │
        ▼             ▼             ▼             ▼             ▼
┌──────────────────────────────────────────────────────────────────────┐
│                       ~/.repo-knowledge/                             │
│                                                                      │
│  _registry.md          _repos/              <project>/               │
│  (project list)        (git clones)         _meta.md                 │
│                                             _index.md                │
│                                             docs/                    │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Three-Phase Data Flow

### Phase 1: Create (rk-create)

```
git clone → scan files → Indexer Agent → per-function docs → _index.md → _meta.md → _registry.md
```

1. **Clone**: Repo cloned to `~/.repo-knowledge/_repos/<project>/`
2. **Scan**: Glob finds all code files; skips directories: `node_modules`, `.git`, `dist`, `build`, `vendor`, `__pycache__`, `.next`, `.nuxt`, `target`, `out`, `coverage`; and skips files that are: test files, config-only files, generated code, or type-definition-only files
3. **Index**: Indexer Agent reads each file, extracts functions/classes, generates Markdown doc for each
4. **Organize**: Docs saved to `~/.repo-knowledge/<project>/docs/<source-path>/<kebab-name>.md`
5. **Register**: `_index.md` (one-line per entry) + `_meta.md` (metadata) + `_registry.md` (global list)

### Phase 2: Search (rk-search)

```
query → _index.md → [alias match | semantic match] → verify cached doc → [hit | fallback to source]
```

Searcher Agent follows a strict 4-layer strategy — layers are tried in order, never skipped:

| Layer | Strategy | Condition to advance |
|-------|----------|---------------------|
| 1 | Alias match in `_index.md` | No alias match → Layer 2 |
| 2 | Semantic ranking of `_index.md` descriptions | Pick best candidate → Layer 3 |
| 3 | Read & verify cached doc | Doc doesn't answer query → try next (max 3) → Layer 4 |
| 4 | Grep source code, generate new doc, save to cache | Always returns a result |

### Phase 3: Update (rk-update)

```
read last_commit → git pull → git diff → process changed files only → update _meta.md
```

Only files that changed between `last_commit` and `HEAD` are reprocessed. Unchanged files keep their cached docs untouched.

---

## Cache Mechanism

Each function/class doc is stored as an individual `.md` file, mirroring the source directory structure:

```
source:  src/auth/validate-token.ts  →  function validateToken()
cache:   ~/.repo-knowledge/project/docs/src/auth/validate-token.md
```

Doc frontmatter tracks source and cache date:

```markdown
---
cached: 2026-04-14
source: src/auth/validate-token.ts
---
```

---

## Alias Learning

Each `_index.md` entry maintains an `aliases` list (max 10) recording past queries that successfully matched it.

```markdown
- [validate-token](docs/src/auth/validate-token.md) — Validate JWT token
  aliases: [token validation, JWT check, how to verify user identity, validate token]
```

**LRU policy:**

New alias is prepended (most-recent-first). When full (10), the last entry is evicted. If user says "wrong result", the alias is removed.

---

## Agent Roles

| Agent | File | Responsibilities |
|-------|------|-----------------|
| Indexer | `agents/indexer.md` | Scans code, generates docs, builds index |
| Searcher | `agents/searcher.md` | Executes 4-layer search, updates aliases |
| Update | `skills/rk-update/SKILL.md` (inline definition) | Processes changed files, incrementally rebuilds docs |

All three agents are dispatched on-demand and do not run persistently. Indexer and Searcher have dedicated agent definition files; the Update Agent is defined as an inline prompt inside the rk-update Skill.
