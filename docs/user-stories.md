# User Stories & Use Cases

Each User Story from [epic.md](epic.md) is expanded here into concrete Use Cases with preconditions, main flow, alternate flows, and postconditions.

---

## US-1.1 — Index a new repository / 索引新仓库

### UC-1.1.1: Index a public GitHub repo

**前置条件 / Preconditions**
- Claude Code is running with repo-knowledge plugin loaded
- User has network access to clone the repository

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-create https://github.com/org/project.git`
2. System clones the repo to `~/.repo-knowledge/_repos/project/`
3. System creates cache directory `~/.repo-knowledge/project/docs/`
4. Indexer Agent scans all code files (*.ts, *.py, *.go, etc.)
5. For each function/class found, Agent generates a documentation entry and saves to cache
6. System builds `_index.md` with one-line summaries for all entries
7. System saves `_meta.md` with git URL, creation date, last commit hash, and doc count
8. System appends project row to `~/.repo-knowledge/_registry.md`
9. System reports: "Indexed `project`: N docs generated"

**异常流程 / Alternate Flows**
- *Network failure during clone*: System reports clone error; no partial state is written
- *Empty repository*: System reports "No code files found" and skips index creation
- *Project already exists in registry*: System warns user and suggests using `rk-update` instead

**后置条件 / Postconditions**
- `~/.repo-knowledge/project/_index.md` exists with all doc entries
- Project row exists in `_registry.md`
- Knowledge base is ready for `rk-search`

---

## US-2.1 — Search with natural language / 自然语言搜索

### UC-2.1.1: Search finds answer in cache

**前置条件 / Preconditions**
- At least one knowledge base exists in `_registry.md`
- `_index.md` has been built for the target project

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-search 如何验证用户身份`
2. System identifies target project (auto-selects if only one; prompts if multiple)
3. Searcher Agent reads `_index.md`
4. Agent scans aliases — no direct alias match found
5. Agent uses semantic understanding to rank index entries; selects `validate-token` as best match
6. Agent reads cached doc for `validate-token`
7. Agent judges: doc answers the query ✓
8. Agent prepends `"如何验证用户身份"` to `validate-token`'s aliases in `_index.md`
9. System returns cached doc content to user

**异常流程 / Alternate Flows**
- *Alias match at step 4*: Skip semantic ranking, go directly to verify cached doc
- *First candidate fails verification*: Try next candidate (up to 3 attempts)
- *All 3 candidates fail*: Proceed to UC-2.1.2 (source code fallback)

**后置条件 / Postconditions**
- User receives relevant documentation
- New alias added to `_index.md` for faster future lookups

### UC-2.1.2: Search falls back to source code

**前置条件 / Preconditions**
- Knowledge base exists but cache doesn't contain a relevant entry

**主流程 / Main Flow**
1. After 3 failed cache candidates, Agent switches to source code search
2. Agent uses Glob to find all code files in `~/.repo-knowledge/_repos/project/`
3. Agent uses Grep with 3+ synonym variations of the query
4. Agent reads matching file sections for context
5. Agent generates new documentation entry in standard format
6. Agent saves new `.md` file to cache under correct directory
7. Agent appends new entry to `_index.md` with query as first alias
8. System returns newly generated doc to user

**后置条件 / Postconditions**
- New doc cached; future searches for same concept hit cache at Layer 1 or 2

---

## US-3.1 — Incremental update / 增量更新

### UC-3.1.1: Update after new commits

**前置条件 / Preconditions**
- Project exists in registry with a recorded `last_commit` hash
- Remote repository has new commits since last update

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-update project`
2. System reads `last_commit` from `~/.repo-knowledge/project/_meta.md`
3. System runs `git pull` in `~/.repo-knowledge/_repos/project/`
4. System runs `git diff --name-only <last_commit>..HEAD` to get changed file list
5. Update Agent processes each changed file:
   - **Modified**: regenerates docs for all functions/classes in that file
   - **Added**: generates new docs, appends entries to `_index.md`
   - **Deleted**: removes cached `.md` files, removes entries from `_index.md`
6. System updates `_meta.md` with new `last_commit`, `last_update`, `total_docs`
7. System updates `_registry.md` row with new date and doc count
8. System reports summary: modified/added/removed counts

**异常流程 / Alternate Flows**
- *No new commits*: System reports "Already up to date" and exits
- *Project not in registry*: System suggests using `rk-create` instead

**后置条件 / Postconditions**
- Cache reflects current state of the codebase
- `_meta.md` has updated `last_commit` hash

---

## US-4.2 — Delete a knowledge base / 删除知识库

### UC-4.2.1: Delete with confirmation

**前置条件 / Preconditions**
- Project exists in registry

**主流程 / Main Flow**
1. User types `/repo-knowledge:rk-delete project`
2. System asks: "Are you sure you want to delete the knowledge base for `project`? This will remove all cached documentation."
3. User confirms
4. System deletes `~/.repo-knowledge/project/` (entire cache directory)
5. System deletes `~/.repo-knowledge/_repos/project/` (cloned repo)
6. System removes project row from `_registry.md`
7. System confirms: "Knowledge base for `project` has been deleted."

**异常流程 / Alternate Flows**
- *User declines confirmation*: System cancels, nothing is deleted
- *Project not in registry*: System reports "No knowledge base found for `project`"

**后置条件 / Postconditions**
- No files remain for the project under `~/.repo-knowledge/`
- Project row removed from `_registry.md`
