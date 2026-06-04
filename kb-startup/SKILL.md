---
name: kb-startup
description: Use when starting a new session, switching context, or needing to recall prior work. Loads active tasks, recent context, and relevant knowledge from the personal Obsidian knowledge base.
---

# KB Startup

Load memory from the personal knowledge base at session start.

## Vault path

```
/Users/weishengdong/Library/Mobile Documents/iCloud~md~obsidian/Documents/丑丑老板的知识库
```

## Read sequence (always in order)

```bash
# 1. Index — active tasks + high-frequency links
obsidian read file="MemoryIndex"

# 2. Hot — recent 1-2 week context snapshot
obsidian read file="Hot"

# 3. Active tasks — read each file linked in MemoryIndex's "当前活跃任务" section
obsidian read file="<task-name>"
```

Stop after step 3. Do NOT scan the whole vault.

## When to go deeper

| Situation | Action |
|-----------|--------|
| MemoryIndex has a direct link | Jump to that file |
| Topic keyword unclear | `obsidian search query="keyword" limit=5` |
| Looking for past decisions | `obsidian search query="[decision]" limit=5` |
| Looking for incident root causes | `obsidian search query="[root_cause]" limit=5` |

## Output

After reading, briefly summarize:
- Current active task(s)
- Key decisions or constraints in scope
- Any blockers or next steps from Hot.md

Do not dump raw note content at the user. Synthesize.
