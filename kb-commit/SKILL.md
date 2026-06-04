---
name: kb-commit
description: Use at the end of a valuable session, after a key decision is made, after a bug is fixed, or when the user asks to save/remember something. Writes structured knowledge back to the personal Obsidian knowledge base.
---

# KB Commit

Write valuable knowledge from the current session back to the knowledge base.

## Vault path

```
/Users/weishengdong/Library/Mobile Documents/iCloud~md~obsidian/Documents/丑丑老板的知识库
```

## Step 1 — Classify what to write

| Content type | Target directory |
|---|---|
| Active task progress / decisions | `00-Agent/Active/<task>.md` (rewrite, not append) |
| Recent context snapshot | `00-Agent/Hot.md` (append, keep under 800 chars) |
| Confirmed design decision | `40-决策/` (new ADR file) |
| Technical system doc | `20-系统/` |
| Incident post-mortem | `60-故障/` |
| Concept note | `10-概念/` |
| Project process doc | `50-项目/` |

## Step 2 — Search before writing

```bash
obsidian search query="<topic>" limit=5
```

If note exists: append an Observation or update Relations. Do NOT duplicate.
If not: create new note.

## Step 3 — Write using structured format

**Observations** (atomic facts):
```
- [category] fact content #optional-tag
```

Common categories: `[decision]` `[finding]` `[pitfall]` `[next]` `[blocker]` `[mechanism]` `[root_cause]` `[fix]`

**Relations** (directed edges):
```
- relation_type [[Target Note]]
```

Common types: `relates_to` `implements` `depends_on` `part_of` `supersedes`

## Step 4 — Update Hot.md and Log.md

```bash
# Append to Hot.md if context is fresh and important
obsidian append file="Hot" content="\n## <date> <topic>\n- <key finding>"

# Always log what was written
obsidian append file="Log" content="\n- <action> [[<note>]]"
```

## Step 5 — Output Memory Commit proposal

Before writing, output this block and wait for confirmation:

```
建议写入：
- [位置] [操作类型：追加/新建/更新] [内容摘要]

不建议写入：
- [原因]
```

Do NOT write automatically without showing the proposal first.

## What NOT to write

- Passwords, tokens, private keys, cookies, sessions
- Unverified guesses or temporary command output
- Raw logs, unsanitized production data
- One-off chat content with no reuse value
