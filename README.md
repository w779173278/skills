# skills

自己使用的一些 Claude Code Skills 集合。

## 安装

把任意 skill 目录复制到 `~/.claude/skills/` 即可在 Claude Code 中通过 `/<skill-name>` 调用。

```bash
# 示例：安装 clean-code
cp -r clean-code ~/.claude/skills/
```

## 当前 Skills

| 名称 | 描述 |
|---|---|
| [clean-code](./clean-code) | 基于 Robert C. Martin《代码整洁之道》的代码编写指南，覆盖命名、函数、注释、格式化、对象与数据结构、错误处理、边界、单元测试、类、系统、并发等主题，提供可直接对照的清单与示例。 |

## Skill 结构

每个 skill 是一个目录，包含 `SKILL.md`，frontmatter 至少需要 `name` 与 `description`：

```markdown
---
name: skill-name
description: 一句话描述用途，用于触发匹配
---

# 正文…
```
