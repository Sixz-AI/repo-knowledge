# Epic & Features

## Epic

**EN:** Enable AI coding assistants to deeply understand any codebase without reading every file — by building a structured, queryable knowledge graph that captures code semantics, relationships, and design rationale at the function level.

**CN:** 让 AI 编程助手无需逐文件阅读，就能深度理解任意代码库——通过构建结构化、可查询的知识图谱，在函数级别捕捉代码语义、关系与设计理由。

---

## Feature 1: Knowledge Base Creation / 知识库创建

**EN:** Index any Git repository by cloning it locally, scanning all code files, and generating per-function/class documentation into a searchable cache.

**CN:** 通过本地克隆 Git 仓库、扫描所有代码文件，为每个函数/类生成文档并存入可搜索缓存，完成任意仓库的知识库索引。

### User Stories

**US-1.1**
> As a developer, I want to index a new Git repository with a single command, so that I can immediately start querying its code without reading every file.
>
> 作为开发者，我希望用一条命令就能索引新的 Git 仓库，以便无需逐文件阅读就能立即开始查询代码。

**US-1.2**
> As a developer, I want each function and class to have its own documentation entry, so that I can get precise, focused answers rather than file-level summaries.
>
> 作为开发者，我希望每个函数和类都有独立的文档条目，以便获得精准、聚焦的答案而非文件级摘要。

**US-1.3**
> As a developer, I want the knowledge base to be registered in a central registry, so that I can manage multiple codebases and know what's available.
>
> 作为开发者，我希望知识库被注册到统一的注册表中，以便管理多个代码库并了解已有哪些知识库。

---

## Feature 2: Semantic Search / 语义搜索

**EN:** Query the knowledge base using natural language in any language, with an Agent that semantically matches queries to documentation even when exact keywords don't match.

**CN:** 使用任意语言的自然语言查询知识库，Agent 能在关键词不精确匹配的情况下，通过语义理解将查询与文档关联。

### User Stories

**US-2.1**
> As a developer, I want to search the codebase using natural language (including Chinese), so that I can find what I need without knowing exact function names.
>
> 作为开发者，我希望用自然语言（包括中文）搜索代码库，以便无需知道准确的函数名就能找到所需内容。

**US-2.2**
> As a developer, I want the search to learn from my queries over time, so that repeated searches become faster and more accurate.
>
> 作为开发者，我希望搜索能随时间从我的查询中学习，以便重复搜索变得更快更准确。

**US-2.3**
> As a developer, I want the system to fall back to searching source code when the cache doesn't have an answer, so that I never get a "not found" result for code that exists.
>
> 作为开发者，我希望缓存没有答案时系统能回退到搜索源码，确保已存在的代码不会出现"未找到"的结果。

---

## Feature 3: Incremental Updates / 增量更新

**EN:** Keep the knowledge base in sync with the codebase by detecting changed files via git history and regenerating only the affected documentation.

**CN:** 通过 git 历史检测变更文件，仅重新生成受影响的文档，使知识库与代码库保持同步。

### User Stories

**US-3.1**
> As a developer, I want to update the knowledge base after new commits without re-indexing the entire repo, so that updates are fast regardless of repo size.
>
> 作为开发者，我希望在新提交后更新知识库时无需重新索引整个仓库，使更新速度与仓库大小无关。

**US-3.2**
> As a developer, I want deleted files to be automatically removed from the knowledge base, so that I don't get results for code that no longer exists.
>
> 作为开发者，我希望已删除文件自动从知识库中移除，避免为不再存在的代码返回结果。

---

## Feature 4: Knowledge Base Management / 知识库管理

**EN:** List all indexed codebases and delete ones that are no longer needed.

**CN:** 列出所有已索引的代码库，并删除不再需要的知识库。

### User Stories

**US-4.1**
> As a developer, I want to see all my indexed codebases in one place, so that I know what's available and when it was last updated.
>
> 作为开发者，我希望在一处查看所有已索引的代码库，了解有哪些知识库以及最后更新时间。

**US-4.2**
> As a developer, I want to delete a knowledge base and all its cached data with one command, so that I can free up disk space when a project is no longer relevant.
>
> 作为开发者，我希望用一条命令删除知识库及其所有缓存数据，以便在项目不再相关时释放磁盘空间。
