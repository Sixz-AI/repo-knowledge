# Getting Started

## Prerequisites

- Claude Code CLI installed

## Installation

Run these two commands inside Claude Code:

```
/plugin marketplace add Sixz-AI/repo-knowledge
/plugin install repo-knowledge@repo-knowledge
```

The SessionStart hook auto-creates the data directory on first launch.

---

## Step 1: Index Your First Repo

```
/repo-knowledge:rk-create https://github.com/org/my-project.git
```

SSH format also works:
```
/repo-knowledge:rk-create git@github.com:org/my-project.git
```

Expected output:
```
Cloning into 'my-project'...
[Indexer Agent] Scanning 42 files...
[Indexer Agent] Generated 128 docs
Knowledge base created: my-project (128 docs)
```

---

## Step 2: Search

```
/repo-knowledge:rk-search how does authentication work
/repo-knowledge:rk-search 认证逻辑在哪里
```

Expected output:
```
Searching my-project...
[Layer 2 semantic match] → authenticate-user.md

## authenticateUser
**File:** src/auth/authenticate.ts
...
```

---

## Step 3: Update After New Commits

```
/repo-knowledge:rk-update my-project
```

Expected output:
```
Pulling latest changes...
Changed files: 3
  Modified: src/auth/authenticate.ts (2 docs regenerated)
  Added: src/utils/hash.ts (1 new doc)
  Deleted: src/auth/legacy.ts (2 docs removed)
Updated: my-project (129 docs, last commit: a1b2c3d)
```

---

## Step 4: Save Personal Notes

Store anything you know by heart — commands, tips, workflows — so you can search it later:

```
/repo-knowledge:rk-memo git undo last commit: git reset --soft HEAD~1
/repo-knowledge:rk-memo kubectl list all pods: kubectl get pods -A --watch
```

Retrieve it any time:
```
/repo-knowledge:rk-search undo commit
```

Personal notes live in a special `_personal` project (no git repo needed) and sync across machines just like indexed codebases.

---

## Step 5: Sync Across Machines (Optional)

Prepare a **private** git repository to store your knowledge base, then:

**On the first machine:**
```
/repo-knowledge:rk-remote-init git@github.com:yourname/my-knowledge.git
```

**On every other machine (after installing the plugin):**
```
/repo-knowledge:rk-remote-init git@github.com:yourname/my-knowledge.git
/repo-knowledge:rk-pull
```

From this point, `rk-update` automatically pulls before updating and pushes afterwards — no manual sync needed.

To push or pull manually:
```
/repo-knowledge:rk-push
/repo-knowledge:rk-pull
```

---

## Step 6: Manage Knowledge Bases

List all:
```
/repo-knowledge:rk-list
```

Delete:
```
/repo-knowledge:rk-delete my-project
```

---

## Tips

- Search supports any language, including mixed CN+EN
- After first search, similar queries find it faster (progressive alias learning)
- Large repos (1000+ files) may take a few minutes to index
- Always use a **private** remote repository — docs contain code snippets
