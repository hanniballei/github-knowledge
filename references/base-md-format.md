# BASE.md Format Reference

BASE.md is the knowledge index file stored in the user's configured `KNOWLEDGE_BASE_PATH`.
It lists all locally saved GitHub repos with their summaries.

## File Location

```
$KNOWLEDGE_BASE_PATH/BASE.md
```

Create it if it doesn't exist when adding the first repo entry.

## Full Format

```markdown
# GitHub Knowledge Base

> Local path: /absolute/path/to/KNOWLEDGE_BASE_PATH
> Last updated: YYYY-MM-DD

---

## [owner/repo-name](https://github.com/owner/repo-name)

**Local**: `$KNOWLEDGE_BASE_PATH/owner/repo-name`
**Updated**: YYYY-MM-DD

owner/repo-name 是一个... [one concise paragraph summary in the user's language]

---

## [owner/another-repo](https://github.com/owner/another-repo)

**Local**: `$KNOWLEDGE_BASE_PATH/owner/another-repo`
**Updated**: YYYY-MM-DD

another-repo 提供了... [summary paragraph]

---
```

## Rules

1. **One entry per repo** — identified by `owner/repo-name`
2. **Most recently updated at the top** — newest additions or updates go first
3. **Summary language** — write in the same language the user used when requesting the download
4. **Local path must be absolute** — resolve `$KNOWLEDGE_BASE_PATH` to the actual path
5. **Update in-place** — when refreshing a summary, replace the existing entry, don't duplicate it

## Summary Content Guidelines

A good summary paragraph covers:
- What the project is / does (1 sentence)
- Core use case or problem it solves (1 sentence)
- Key technologies or notable features (1 sentence)
- Current status if notable (active, archived, widely used, etc.)

Keep it to 3–5 sentences. Aim for someone to understand the repo's value in 10 seconds.

## Example Entry

```markdown
## [langchain-ai/langchain](https://github.com/langchain-ai/langchain)

**Local**: `/home/user/github-knowledge/langchain-ai/langchain`
**Updated**: 2026-02-18

langchain 是一个用于构建大语言模型（LLM）应用的 Python/JavaScript 框架，提供链式调用、
记忆管理、工具集成和 Agent 等核心抽象。它解决了将 LLM 与外部数据源、工具和工作流集成的
复杂性，支持 OpenAI、Anthropic、本地模型等多种后端。该项目活跃维护，拥有庞大的社区生态，
是目前最流行的 LLM 应用开发框架之一。
```
