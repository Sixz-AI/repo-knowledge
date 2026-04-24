# 快速上手

## 前置条件

- Claude Code CLI 已安装

## 安装

在 Claude Code 中执行以下两条命令：

```
/plugin marketplace add Sixz-AI/repo-knowledge
/plugin install repo-knowledge@repo-knowledge
```

首次启动时，SessionStart Hook 会自动创建数据目录。

---

## Step 1: 索引第一个仓库

```
/repo-knowledge:rk-create https://github.com/org/my-project.git
```

SSH 格式也支持：
```
/repo-knowledge:rk-create git@github.com:org/my-project.git
```

预期输出：
```
Cloning into 'my-project'...
[Indexer Agent] Scanning 42 files...
[Indexer Agent] Generated 128 docs
Knowledge base created: my-project (128 docs)
```

---

## Step 2: 搜索

```
/repo-knowledge:rk-search how does authentication work
/repo-knowledge:rk-search 认证逻辑在哪里
```

预期输出：
```
Searching my-project...
[Layer 2 semantic match] → authenticate-user.md

## authenticateUser
**File:** src/auth/authenticate.ts
...
```

---

## Step 3: 新提交后更新

```
/repo-knowledge:rk-update my-project
```

预期输出：
```
Pulling latest changes...
Changed files: 3
  Modified: src/auth/authenticate.ts (2 docs regenerated)
  Added: src/utils/hash.ts (1 new doc)
  Deleted: src/auth/legacy.ts (2 docs removed)
Updated: my-project (129 docs, last commit: a1b2c3d)
```

---

## Step 4: 保存个人笔记

把脑子里常用的东西存下来——命令、技巧、个人工作流——随时搜索：

```
/repo-knowledge:rk-memo git 撤销最后一次 commit: git reset --soft HEAD~1
/repo-knowledge:rk-memo kubectl 查看所有 pod: kubectl get pods -A --watch
```

之后直接搜索：
```
/repo-knowledge:rk-search 撤销 commit
```

个人笔记存放在特殊的 `_personal` 项目中（不需要 git 仓库），和代码库知识一起跨机器同步。

---

## Step 5: 跨机器同步（可选）

准备一个**私有** git 仓库用于存放知识库，然后：

**第一台机器：**
```
/repo-knowledge:rk-remote-init git@github.com:yourname/my-knowledge.git
```

**其他机器（安装插件后）：**
```
/repo-knowledge:rk-remote-init git@github.com:yourname/my-knowledge.git
/repo-knowledge:rk-pull
```

配置完成后，`rk-update` 每次运行会自动 pull（开头）和 push（结尾），无需手动同步。

手动触发同步：
```
/repo-knowledge:rk-push
/repo-knowledge:rk-pull
```

---

## Step 6: 管理知识库

列出所有知识库：
```
/repo-knowledge:rk-list
```

删除知识库：
```
/repo-knowledge:rk-delete my-project
```

---

## 提示

- 搜索支持任意语言，中英混用均可
- 首次搜索某个概念后，下次用相似词会更快找到（渐进式 alias 学习）
- 大型仓库（1000+ 文件）索引可能需要几分钟
- 远端仓库必须设为**私有**——文档中包含被索引项目的代码片段
