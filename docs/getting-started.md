# Getting Started / 快速上手

## 前置条件 / Prerequisites

- Claude Code CLI 已安装 / Claude Code CLI installed
- 已安装插件支持 / Plugin support available

## 安装 / Installation

```bash
# 进入 repo-knowledge 目录
cd ~/Documents/code/ai/repo-knowledge

# 以插件模式启动 Claude Code / Launch Claude Code with plugin
claude --plugin-dir .
```

首次启动时，SessionStart Hook 会自动创建数据目录：

The SessionStart hook auto-creates the data directory on first launch:

```bash
mkdir -p ~/.repo-knowledge/_repos
touch -a ~/.repo-knowledge/_registry.md
```

---

## Step 1: 索引第一个仓库 / Index Your First Repo

```
/repo-knowledge:rk-create https://github.com/org/my-project.git
```

预期输出 / Expected output:
```
Cloning into 'my-project'...
[Indexer Agent] Scanning 42 files...
[Indexer Agent] Generated 128 docs
Knowledge base created: my-project (128 docs)
```

SSH 格式也支持 / SSH format also works:
```
/repo-knowledge:rk-create git@github.com:org/my-project.git
```

---

## Step 2: 第一次搜索 / Your First Search

```
/repo-knowledge:rk-search how does authentication work
```

或用中文 / Or in Chinese:
```
/repo-knowledge:rk-search 认证逻辑在哪里
```

预期输出 / Expected output:
```
Searching my-project...
[Layer 2 semantic match] → authenticate-user.md

## authenticateUser
**File:** src/auth/authenticate.ts
...
```

---

## Step 3: 新提交后更新 / Update After New Commits

当仓库有新提交时 / When the repo has new commits:

```
/repo-knowledge:rk-update my-project
```

预期输出 / Expected output:
```
Pulling latest changes...
Changed files: 3
  Modified: src/auth/authenticate.ts (2 docs regenerated)
  Added: src/utils/hash.ts (1 new doc)
  Deleted: src/auth/legacy.ts (2 docs removed)
Updated: my-project (129 docs, last commit: a1b2c3d)
```

---

## Step 4: 管理知识库 / Manage Knowledge Bases

列出所有知识库 / List all:
```
/repo-knowledge:rk-list
```

预期输出 / Expected:
```
| Project    | Git URL                              | Last Update | Docs |
|------------|--------------------------------------|-------------|------|
| my-project | https://github.com/org/my-project... | 2026-04-14  | 129  |
```

删除知识库 / Delete:
```
/repo-knowledge:rk-delete my-project
```

系统会要求确认后再删除 / System asks for confirmation before deleting.

---

## 提示 / Tips

- 搜索支持任意语言，中英混用均可 / Search supports any language, including mixed CN+EN
- 首次搜索某个概念后，下次用相似词会更快找到 / After first search, similar queries find it faster
- 大型仓库（1000+ 文件）索引可能需要几分钟 / Large repos (1000+ files) may take a few minutes to index
