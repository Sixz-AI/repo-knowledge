# 架构与原理

## 系统组件

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

## 三阶段数据流

### 阶段 1：Create（rk-create）

```
git clone → scan files → Indexer Agent → per-function docs → _index.md → _meta.md → _registry.md
```

1. **Clone**：仓库克隆至 `~/.repo-knowledge/_repos/<project>/`
2. **Scan**：Glob 扫描所有代码文件；跳过目录：`node_modules`、`.git`、`dist`、`build`、`vendor`、`__pycache__`、`.next`、`.nuxt`、`target`、`out`、`coverage`；以及跳过：测试文件、仅含配置的文件、生成代码、仅含类型定义的文件
3. **Index**：Indexer Agent 读取每个文件，提取函数/类，为每个生成 Markdown 文档
4. **Organize**：文档保存至 `~/.repo-knowledge/<project>/docs/<source-path>/<kebab-name>.md`
5. **Register**：`_index.md`（每条目一行）+ `_meta.md`（元数据）+ `_registry.md`（全局列表）

### 阶段 2：Search（rk-search）

```
query → _index.md → [alias match | semantic match] → verify cached doc → [hit | fallback to source]
```

Searcher Agent 遵循严格的 4 层策略——按顺序逐层尝试，不可跳过：

| 层级 | 策略 | 推进条件 |
|------|------|---------|
| 1 | 在 `_index.md` 中匹配别名 | 无别名匹配 → 第 2 层 |
| 2 | 对 `_index.md` 描述进行语义排名 | 选出最佳候选 → 第 3 层 |
| 3 | 读取并验证缓存文档 | 文档无法回答查询 → 尝试下一个（最多 3 次）→ 第 4 层 |
| 4 | Grep 源代码，生成新文档，保存至缓存 | 始终返回结果 |

### 阶段 3：Update（rk-update）

```
read last_commit → git pull → git diff → process changed files only → update _meta.md
```

仅重新处理 `last_commit` 与 `HEAD` 之间发生变更的文件。未变更的文件保留其缓存文档不变。

---

## 缓存机制

每个函数/类的文档单独存储为一个 `.md` 文件，路径与源文件目录结构对应：

```
source:  src/auth/validate-token.ts  →  function validateToken()
cache:   ~/.repo-knowledge/project/docs/src/auth/validate-token.md
```

文档头部包含 frontmatter，用于追踪来源和缓存日期：

```markdown
---
cached: 2026-04-14
source: src/auth/validate-token.ts
---
```

---

## 别名学习机制

`_index.md` 中每个条目维护一个 aliases 列表（最多 10 个），记录历史上成功匹配该条目的查询词。

```markdown
- [validate-token](docs/src/auth/validate-token.md) — Validate JWT token（验证 JWT 令牌）
  aliases: [如何验证用户身份, token validation, JWT check, 验证令牌]
```

**LRU 策略：**
- 新别名插入列表头部（最近优先）
- 满 10 个时，删除最末尾（最久未用）的别名
- 用户明确反馈"错误"时，移除该别名

---

## Agent 角色

| Agent | 文件 | 职责 |
|-------|------|------|
| Indexer | `agents/indexer.md` | 扫描代码文件，生成文档，构建索引 |
| Searcher | `agents/searcher.md` | 执行4层搜索策略，更新别名 |
| Update | `skills/rk-update/SKILL.md`（内联定义）| 处理变更文件，增量重建文档 |

三个 Agent 均由对应 Skill 按需派发，不持久运行。Indexer 和 Searcher 有独立的 Agent 定义文件；Update Agent 以内联 prompt 形式定义在 rk-update Skill 中。
