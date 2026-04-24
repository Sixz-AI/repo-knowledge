# Skill API 参考

所有 Skill 均通过 Claude Code 的 `/` 命令触发。

---

## rk-create

**触发：** `/repo-knowledge:rk-create <git-url>`

| 参数 | 类型 | 说明 |
|------|------|------|
| `git-url` | string (required) | HTTPS 或 SSH 格式的 Git 仓库地址 |

**行为：**
1. 从 URL 推断项目名（取最后一段，去掉 `.git`）
2. 克隆仓库到 `~/.repo-knowledge/_repos/<project>/`
3. 派发 Indexer Agent 扫描所有代码文件并生成文档
4. 保存 `_meta.md`、`_index.md`，注册到 `_registry.md`

**示例：**
```
/repo-knowledge:rk-create https://github.com/anthropics/anthropic-sdk-python.git
/repo-knowledge:rk-create git@github.com:org/project.git
```

**注意：** 项目名已存在时会提示使用 `rk-update`。私有仓库需要配置 SSH key 或 token。

---

## rk-search

**触发：** `/repo-knowledge:rk-search <query>`

| 参数 | 类型 | 说明 |
|------|------|------|
| `query` | string (required) | 自然语言查询，任意语言 |

**行为：**
1. 读取 `_registry.md`，确定搜索目标（单项目自动选择，多项目时提示选择）
2. 派发 Searcher Agent 执行 4 层搜索策略
3. 返回匹配的文档内容
4. 将查询词作为别名写入 `_index.md`

**4 层搜索策略：**
1. **别名匹配** — 在已有 alias 中精确/近似匹配
2. **语义匹配** — 用语言理解能力在 `_index.md` 描述中找最相关条目
3. **验证** — 读取候选文档确认是否真正回答了查询（最多 3 次）
4. **源码兜底** — 直接搜索源文件，生成并缓存新文档

**示例：**
```
/repo-knowledge:rk-search how does authentication work
/repo-knowledge:rk-search 窗口大小
/repo-knowledge:rk-search JWT token validation
```

**注意：** 支持跨语言查询（中文查询可匹配英文文档）。每次成功搜索后 alias 自动更新（LRU，最多 10 个）。

---

## rk-update

**触发：** `/repo-knowledge:rk-update <project-name>`

| 参数 | 类型 | 说明 |
|------|------|------|
| `project-name` | string (required) | 已注册的项目名 |

**行为：**
1. 如果配置了 `_remote.md`，先自动从远端 pull
2. 如果 `_repos/<project>` 不存在（换机器后），自动从 `_meta.md` 读取 `git_url` 重新 clone
3. 如果项目是个人知识库（`type: personal`），跳过并提示使用 `rk-memo`
4. 对源码仓库执行 `git pull`，对比 `last_commit` 找出变更文件
5. 对变更文件增量处理：修改重建、新增添加、删除清理
6. 更新 `_meta.md` 和 `_registry.md`
7. 如果配置了 `_remote.md`，自动 push 到远端

**示例：**
```
/repo-knowledge:rk-update my-project
```

---

## rk-list

**触发：** `/repo-knowledge:rk-list`

**行为：** 读取并格式化显示 `~/.repo-knowledge/_registry.md`。个人知识库在 Git URL 列显示 `(personal)`。

**示例：**
```
/repo-knowledge:rk-list
```

预期输出：
```
| Project    | Git URL                    | Last Update | Docs |
|------------|----------------------------|-------------|------|
| my-project | https://github.com/org/... | 2026-04-14  | 129  |
| _personal  | (personal)                 | 2026-04-24  |  12  |
```

---

## rk-delete

**触发：** `/repo-knowledge:rk-delete <project-name>`

| 参数 | 类型 | 说明 |
|------|------|------|
| `project-name` | string (required) | 要删除的项目名 |

**行为：**
1. 向用户确认是否删除
2. 删除 `~/.repo-knowledge/<project>/`（缓存目录）
3. 删除 `~/.repo-knowledge/_repos/<project>/`（个人知识库跳过此步）
4. 从 `_registry.md` 移除对应行

**注意：** 操作不可逆，删除前系统会要求确认。

---

## rk-memo

**触发：** `/repo-knowledge:rk-memo <标题>: <内容>`

| 参数 | 类型 | 说明 |
|------|------|------|
| `标题` | string (required) | 这条知识的简短标签 |
| `内容` | string (required) | 要保存的内容（命令、笔记、技巧等） |

**行为：**
1. 如果 `_personal` 项目不存在，自动创建（不需要 git 仓库）
2. 将内容写入 `~/.repo-knowledge/_personal/docs/<kebab-title>.md`
3. 在 `_personal/_index.md` 中新增或更新条目，并提取 alias
4. 如果配置了远端，自动 push

**示例：**
```
/repo-knowledge:rk-memo git 撤销最后一次 commit: git reset --soft HEAD~1
/repo-knowledge:rk-memo kubectl 查看所有 pod: kubectl get pods -A --watch
/repo-knowledge:rk-memo python 虚拟环境: python3 -m venv .venv && source .venv/bin/activate
```

保存后随时搜索：
```
/repo-knowledge:rk-search 撤销 commit
/repo-knowledge:rk-search 查看 pod
```

**注意：** 标题已存在时覆盖该条目。个人知识和代码库知识一起通过 `rk-push/pull` 同步。

---

## rk-remote-init

**触发：** `/repo-knowledge:rk-remote-init <git-url>`

| 参数 | 类型 | 说明 |
|------|------|------|
| `git-url` | string (required) | **私有** git 仓库的 SSH 或 HTTPS 地址 |

**行为：**
1. 跨平台解析 home 目录
2. 将 `~/.repo-knowledge/` 初始化为 git 仓库（如尚未初始化）
3. 创建 `.gitignore`（排除 `_repos/`）
4. 配置 remote 为 `origin`
5. 如果远端已有数据（后续机器）：拉取并合并已有知识
6. 如果远端为空（第一台机器）：将本地知识作为初始状态推送
7. 将远端配置保存到 `_remote.md`

初始化后，`rk-update` 每次运行会自动 pull（开头）和 push（结尾）。

**示例：**
```
/repo-knowledge:rk-remote-init git@github.com:yourname/my-knowledge.git
```

> **安全提示：** 必须使用**私有**仓库。文档中包含被索引项目的代码片段。

---

## rk-push

**触发：** `/repo-knowledge:rk-push`

**行为：**
1. fetch `origin/main`
2. 如果远端有新提交：Agent 对比 diff，按文件类型智能合并
3. commit 合并结果
4. push 到 `origin/main`（被拒绝时最多重试 2 次）

**合并策略：**

| 文件 | 规则 |
|------|------|
| `_index.md` | 同条目 → alias 取并集，保留最新 10 条（本地 alias 优先）；新条目 → 直接追加 |
| `_registry.md` | 按项目名去重，保留 `Last Update` 最新的行 |
| `docs/*.md` | 保留 `cached:` 日期较新的版本 |
| `_meta.md` | 取最大 `last_update`，取最大 `total_docs` |

**示例：**
```
/repo-knowledge:rk-push
```

**注意：** `rk-update` 会自动调用此操作。需要强制同步时可手动触发。

---

## rk-pull

**触发：** `/repo-knowledge:rk-pull`

**行为：** 与 `rk-push` 合并逻辑相同，但只写入本地，**不**推送到远端。在新机器上用来拉取全部已有知识，无需重新索引。

**示例：**
```
/repo-knowledge:rk-pull
```

**注意：**
- 从远端 pull 的项目，`_repos/<project>` 本地不存在。下次运行 `rk-update` 时会自动 clone。
- 任何时候调用都是安全的，不要求远端有新内容。
