---
type: reference
domain: meta
status: stable
tags: [AI, skills, 基础设施]
aliases: ["_技能", Agent技能]
created: 2026-06-03
updated: 2026-06-03
---

# _技能（Skills）

> Agent 可复用的**技能 / 提示词 / 操作模式**库——「怎么做某类任务」的可调用能力。
> 与 `_元数据`、`_记忆` 并列，属知识库的 AI 基础设施。

## 放什么

- **提示词模板**：针对特定任务的高质量 prompt（如「写 ADR」「做故障复盘」）。
- **操作模式**：Agent 处理某类请求的标准步骤/检查清单。
- **领域技能**：结合本库规范的专用能力（如「按 schema 新建笔记」「构建 relates 图谱」）。

## 与 70-手册（Runbook）的区别

| | 70-手册 Runbook | _技能 Skills |
|---|---|---|
| 执行者 | 人 | AI Agent |
| 内容 | 运维 SOP（扩容/切换） | 提示词/Agent 操作模式 |
| 定位 | 知识内容 | AI 基础设施 |

## 格式建议

一技能一文件，frontmatter 标 `type: reference, domain: meta`，正文写清「适用场景 + 步骤/提示词 + 示例」。与 `_记忆` 区互补：技能是「会做什么」，记忆是「记住什么」。参见 [[AGENTS]] 的建笔记流程。
