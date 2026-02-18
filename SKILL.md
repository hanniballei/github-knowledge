---
name: github-knowledge
description: |
  GitHub knowledge base manager for storing, indexing, and querying GitHub repositories locally.
  Maintains a BASE.md index file with summaries of all saved repos.

  Trigger this skill when the user asks a question or makes a request related to:
  - A specific GitHub repository (download, query, analyze, summarize, compare)
  - GitHub issues, PRs, releases, or repository search
  - 仓库、GitHub项目、开源项目 的下载、查询、分析等操作

  Do NOT trigger for unrelated git operations (e.g., "init a local git repo", "git commit").

  Core actions:
  1. Explicit download request → require URL → clone or pull → summarize → update BASE.md
  2. Query + repo exists locally → pull → maybe update summary → answer question
  3. Query + repo NOT local → search GitHub via gh/curl → answer question (do NOT clone)
---

# GitHub Knowledge

## Reading the Knowledge Base Path

At the start of every operation, read the configured path:

```bash
GITHUB_KNOWLEDGE_PATH=$(bash -c 'source ~/.bashrc 2>/dev/null; source ~/.zshrc 2>/dev/null; echo $GITHUB_KNOWLEDGE_PATH')
echo "$GITHUB_KNOWLEDGE_PATH"
```

If empty, ask the user for their desired storage path, then guide them to add to `~/.bashrc` (or `~/.zshrc`):

```bash
export GITHUB_KNOWLEDGE_PATH=/path/chosen/by/user
```

Then reload: `source ~/.bashrc`

---

## Intent Decision Tree

**Determine user intent before taking any action.**

```
User asks about github/repo/仓库?
│
├─ Intent: explicitly download/clone/save a repo?  ──→ [Download Flow]
│   (keywords: 下载, clone, 保存, 收藏, 加入知识库)
│
└─ Intent: ask about / query a repo?               ──→ [Query Flow]
    (keywords: 怎么用, 介绍, 有什么issue, 看看, 了解, 搜索)
```

---

## Download Flow

### 1. Require a repo URL

The user **must** provide a full GitHub URL before cloning. If not given, ask:

> "请提供要下载的 GitHub 仓库链接（格式：https://github.com/owner/repo）"

Do not proceed until a URL is confirmed.

### 2. Resolve owner and repo name

```bash
OWNER=$(echo "$REPO_URL" | awk -F'/' '{print $4}')
REPO=$(echo "$REPO_URL" | awk -F'/' '{print $5}')
LOCAL_PATH="$GITHUB_KNOWLEDGE_PATH/$OWNER/$REPO"
```

### 3. Check if already local

```bash
[ -d "$LOCAL_PATH" ] && echo "EXISTS" || echo "NEW"
```

### 4a. New repo → Clone

For large repos use `--depth 1` to save time and disk space; omit for smaller repos where full history is useful:

```bash
mkdir -p "$GITHUB_KNOWLEDGE_PATH/$OWNER"
git clone --depth 1 "$REPO_URL" "$LOCAL_PATH"
```

Read the repo contents and generate a summary (see [Summarization](#summarization)).
Write a new entry to BASE.md (see [BASE.md format](references/base-md-format.md)).

### 4b. Existing repo → Pull and assess

```bash
cd "$LOCAL_PATH" && git pull
git log --oneline -20
```

If meaningful changes (new features, major fixes, architecture changes): update the summary in BASE.md.
If only minor changes (docs, typos, deps): keep existing summary.

---

## Query Flow

### 1. Resolve the repo

Extract `owner/repo` from the user's message if provided.

If only a repo name is given (no owner):
- Run `gh search repos <name> --sort stars --limit 1` to find the most popular match
- Use that repo and inform the user: "已使用 `owner/repo`，如需其他版本请提供完整链接"

If `gh` is unavailable:
```bash
curl -s "https://api.github.com/search/repositories?q=<name>&sort=stars&per_page=1" \
  | python3 -c "import sys,json; r=json.load(sys.stdin)['items'][0]; print(r['full_name'])"
```

### 2. Check if exists locally

```bash
[ -d "$GITHUB_KNOWLEDGE_PATH/$OWNER/$REPO" ] && echo "EXISTS" || echo "NOT_FOUND"
```

### 3a. Found locally → Pull and answer

```bash
cd "$GITHUB_KNOWLEDGE_PATH/$OWNER/$REPO" && git pull
git log --oneline -20
```

Read local files to answer the user's question.

Check if summary needs updating using the same criteria as Download Flow:
- Meaningful changes (new features, major fixes, architecture changes) → update BASE.md
- Minor changes (docs, typos, deps) → keep existing summary

### 3b. NOT found locally → Search GitHub and answer

Do **not** clone. Use `gh` CLI or curl to fetch information:

```bash
# With gh CLI
gh repo view owner/repo
gh search repos <query> --limit 5
gh search issues --repo owner/repo "<query>"
gh pr list --repo owner/repo --state open --limit 10

# Fallback with curl (60 req/hr unauthenticated)
curl -s "https://api.github.com/repos/owner/repo"
curl -s "https://api.github.com/search/repositories?q=<query>&per_page=5"
curl -s "https://api.github.com/repos/owner/repo/issues?state=open&per_page=10"
curl -s "https://api.github.com/repos/owner/repo/pulls?state=open&per_page=10"
```

Answer the user's question from the fetched data.

---

## Summarization

When generating or updating a repo summary, read in order:
1. `README.md` (primary source)
2. `docs/` directory if present
3. `package.json` / `pyproject.toml` / `Cargo.toml` for tech stack
4. Top-level source structure (`ls -la`)

Write a concise paragraph covering: what it does, the problem it solves, key technologies, and status (active/archived).

Then update BASE.md. See [BASE.md format](references/base-md-format.md).

---

## BASE.md Management

See [BASE.md format](references/base-md-format.md) for the full format specification.

Quick rule: one entry per repo, most recently updated at the top. Update in-place when refreshing; never duplicate entries.
