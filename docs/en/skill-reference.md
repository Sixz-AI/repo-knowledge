# Skill Reference

All Skills are triggered via Claude Code's `/` command syntax.

---

## rk-create

**Trigger:** `/repo-knowledge:rk-create <git-url>`

| Parameter | Type | Description |
|-----------|------|-------------|
| `git-url` | string (required) | HTTPS or SSH Git URL |

**Behavior:**
1. Infers project name from the URL (last segment, stripping `.git`)
2. Clones the repository to `~/.repo-knowledge/_repos/<project>/`
3. Dispatches Indexer Agent to scan all code files and generate documentation
4. Saves `_meta.md`, `_index.md`, and registers the project in `_registry.md`

**Example:**
```
/repo-knowledge:rk-create https://github.com/anthropics/anthropic-sdk-python.git
/repo-knowledge:rk-create git@github.com:org/project.git
```

**Notes:**
- Warns to use `rk-update` if project name already exists
- Private repos require SSH key or token configured

---

## rk-search

**Trigger:** `/repo-knowledge:rk-search <query>`

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | string (required) | Natural language query, any language |

**Behavior:**
1. Reads `_registry.md` to determine the search target (auto-selects for single project, prompts for multiple)
2. Dispatches Searcher Agent to execute a 4-layer search strategy
3. Returns matching document content
4. Writes the query term as an alias into `_index.md`

**Search layers:**
1. **Alias match** — exact/close match against stored aliases
2. **Semantic match** — language-model understanding of `_index.md` descriptions
3. **Verify** — reads the candidate doc and confirms it answers the query (up to 3 attempts)
4. **Source code fallback** — searches source files directly, generates and caches a new doc

**Example:**
```
/repo-knowledge:rk-search how does authentication work
/repo-knowledge:rk-search 窗口大小
/repo-knowledge:rk-search JWT token validation
```

**Notes:**
- Supports cross-language queries (Chinese queries can match English documents)
- Aliases in `_index.md` are updated after each successful search (LRU, max 10)

---

## rk-update

**Trigger:** `/repo-knowledge:rk-update <project-name>`

| Parameter | Type | Description |
|-----------|------|-------------|
| `project-name` | string (required) | Registered project name |

**Behavior:**
1. Auto-pulls from remote if `_remote.md` is configured
2. Auto-clones `_repos/<project>` if missing (e.g. on a new machine after `rk-pull`)
3. Skips with a message if the project is a personal knowledge base (`type: personal`)
4. Runs `git pull` on the source repo, diffs against `last_commit`
5. Incrementally rebuilds docs for modified/added/deleted files
6. Updates `_meta.md` and `_registry.md`
7. Auto-pushes to remote if `_remote.md` is configured

**Example:**
```
/repo-knowledge:rk-update my-project
```

---

## rk-list

**Trigger:** `/repo-knowledge:rk-list`

**Behavior:**
Reads and displays `~/.repo-knowledge/_registry.md`. Personal knowledge bases show `(personal)` in the Git URL column.

**Example:**
```
/repo-knowledge:rk-list
```

Expected output:
```
| Project    | Git URL                    | Last Update | Docs |
|------------|----------------------------|-------------|------|
| my-project | https://github.com/org/... | 2026-04-14  | 129  |
| _personal  | (personal)                 | 2026-04-24  |  12  |
```

---

## rk-delete

**Trigger:** `/repo-knowledge:rk-delete <project-name>`

| Parameter | Type | Description |
|-----------|------|-------------|
| `project-name` | string (required) | Project name to delete |

**Behavior:**
1. Asks the user to confirm deletion
2. Deletes `~/.repo-knowledge/<project>/` (cache directory)
3. Deletes `~/.repo-knowledge/_repos/<project>/` (skipped for personal knowledge bases)
4. Removes the entry from `_registry.md`

**Example:**
```
/repo-knowledge:rk-delete my-project
```

**Notes:** Operation is irreversible. System requires confirmation before deleting.

---

## rk-memo

**Trigger:** `/repo-knowledge:rk-memo <title>: <content>`

| Parameter | Type | Description |
|-----------|------|-------------|
| `title` | string (required) | Short label for this piece of knowledge |
| `content` | string (required) | The knowledge to save (commands, notes, tips, etc.) |

**Behavior:**
1. Creates the `_personal` project if it does not exist (no git repo needed)
2. Writes a doc to `~/.repo-knowledge/_personal/docs/<kebab-title>.md`
3. Adds or updates the entry in `_personal/_index.md` with extracted aliases
4. Auto-pushes if remote is configured

**Example:**
```
/repo-knowledge:rk-memo git undo last commit: git reset --soft HEAD~1
/repo-knowledge:rk-memo kubectl list all pods: kubectl get pods -A --watch
/repo-knowledge:rk-memo python virtual env: python3 -m venv .venv && source .venv/bin/activate
```

After saving, retrieve with:
```
/repo-knowledge:rk-search undo commit
/repo-knowledge:rk-search 查看 pod
```

**Notes:**
- Calling `rk-memo` with an existing title overwrites that entry
- Personal knowledge syncs with all machines via `rk-push/pull`

---

## rk-remote-init

**Trigger:** `/repo-knowledge:rk-remote-init <git-url>`

| Parameter | Type | Description |
|-----------|------|-------------|
| `git-url` | string (required) | SSH or HTTPS URL of a **private** git repository |

**Behavior:**
1. Resolves home directory cross-platform
2. Initializes `~/.repo-knowledge/` as a git repository (if not already)
3. Creates `.gitignore` (excludes `_repos/`)
4. Configures the remote as `origin`
5. If remote already has data (subsequent machine): pulls and merges existing knowledge
6. If remote is empty (first machine): pushes local knowledge as initial state
7. Saves remote config to `_remote.md`

After init, `rk-update` auto-pushes and `rk-update` auto-pulls on every run.

**Example:**
```
/repo-knowledge:rk-remote-init git@github.com:yourname/my-knowledge.git
```

> **Security:** Always use a **private** repository. Docs contain code snippets from your indexed projects.

---

## rk-push

**Trigger:** `/repo-knowledge:rk-push`

**Behavior:**
1. Fetches `origin/main`
2. If remote is ahead: Agent compares diff and merges intelligently per file type
3. Commits merged result
4. Pushes to `origin/main` (retries up to 2× on rejection)

**Merge strategy:**

| File | Rule |
|------|------|
| `_index.md` | Same entry → union aliases, keep newest 10 (local aliases first); new entry → append |
| `_registry.md` | Union by project name, keep row with newest `Last Update` |
| `docs/*.md` | Keep file with newer `cached:` date |
| `_meta.md` | Keep max `last_update`, max `total_docs` |

**Example:**
```
/repo-knowledge:rk-push
```

**Notes:** `rk-update` calls this automatically. Use manually after `rk-memo` or when you want to force a sync.

---

## rk-pull

**Trigger:** `/repo-knowledge:rk-pull`

**Behavior:**
Same merge logic as `rk-push`, but writes result locally only — does **not** push back to remote. Use on a new machine to pull all existing knowledge without re-indexing.

**Example:**
```
/repo-knowledge:rk-pull
```

**Notes:**
- For projects pulled from remote, `_repos/<project>` will not exist locally. It is auto-cloned the next time `rk-update` runs for that project.
- Does not require remote to be ahead — safe to call at any time.
