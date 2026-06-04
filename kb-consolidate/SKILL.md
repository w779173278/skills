---
name: kb-consolidate
description: Use when Hot.md is getting long, Active tasks appear stale, MemoryIndex has dead links, or the user asks to clean up / consolidate memory. Tidies the 00-Agent layer without touching domain notes.
---

# KB Consolidate

Reflective pass over the `00-Agent/` memory layer. Merges duplicates, retires stale content, and keeps the index lean.

## Vault path

```
/Users/weishengdong/Library/Mobile Documents/iCloud~md~obsidian/Documents/丑丑老板的知识库
```

## Phase 1 — Take stock

```bash
obsidian read file="MemoryIndex"
obsidian read file="Hot"
obsidian read file="Log"
# List all Active task files
obsidian search query="type: active" limit=20
```

## Phase 2 — Hot.md cleanup

Rules:
- Keep total under 800 chars
- Drop entries older than 2 weeks unless still relevant
- Convert absolute "next step" items that are done → move to Log.md
- Fix relative dates ("next week") → absolute dates

```bash
# After editing, overwrite Hot.md with cleaned content
obsidian create name="Hot" content="<cleaned content>" overwrite silent
```

## Phase 3 — Active tasks audit

For each file in `00-Agent/Active/`:

| Task status | Action |
|---|---|
| Still in progress | Keep, update `updated:` frontmatter date |
| Completed / merged | Move summary to `50-项目/` or `40-决策/`, delete Active file |
| Stale > 4 weeks, no progress | Archive to `99-归档/` or delete |

Active files must be **rewritten to current state** — no stacking old history at bottom.

## Phase 4 — MemoryIndex cleanup

Rules:
- Max 200 lines
- One line per entry: `- [[Note]] — one-line hook under 150 chars`
- Remove links to deleted/archived notes
- Add newly important notes to "高频知识" section

```bash
obsidian create name="MemoryIndex" content="<trimmed index>" overwrite silent
```

## Phase 5 — Log.md

Log is append-only. Only action: add a consolidation entry.

```bash
obsidian append file="Log" content="\n## <date> 记忆整理\n- 清理 Hot.md，保留 <N> 条\n- 归档 Active 任务 <N> 个\n- MemoryIndex 更新"
```

## Output

Finish with a summary:
- Files touched and what changed
- Active tasks retired
- Lines removed from Hot.md and MemoryIndex
