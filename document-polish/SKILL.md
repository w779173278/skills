---
name: document-polish
description: 标准化 Obsidian 笔记格式，批量润色文档。规范 frontmatter、标题结构、代码块、图片链接和交叉引用，支持多文件并行处理。适用场景：格式化笔记、规范 frontmatter、清理文档，或用户提及"格式化"、"整理文档"、"polish"、"润色"。
priority: P0
origin: user
license: MIT
---

# Document Polish

标准化 Obsidian 笔记格式，批量润色文档。

## 模板即规范

frontmatter 字段和章节结构以 `_元数据/templates/` 下对应模板为唯一标准：

| `type` 字段 | 参照模板 |
|------------|---------|
| concept | [[concept]] |
| system | [[system]] |
| architecture | [[architecture]] |
| adr | [[adr]] |
| incident | [[incident]] |
| project | [[project]] |
| reference / runbook | [[runbook]] |
| moc | [[moc]] |
| meeting | [[meeting]] |
| learning | [[learning]] |
| daily | [[daily]] |

润色前先读取文档的 `type` 字段，再对照对应模板检查 frontmatter 完整性和章节结构。

## 格式修复规则

| 问题 | 修复方式 |
|------|---------|
| 缺少代码语言标记 | 添加 ` ```java ` / ` ```xml ` / ` ```yaml ` 等 |
| 外部图片链接 | 移除 `fynotefile.oss-cn-...` 链接，保留本地图片 |
| 孤立 `image.png` 行 | 移除 |
| 目录/TOC 章节 | 移除 `## 目录` 及其内容 |
| 重复 frontmatter | 保留第一个，移除后续 |
| 标题层级跳跃 | 确保 H1→H2→H3 递进 |
| 中英文混排缺少空格 | 中文与英文/数字间加空格 |
| frontmatter 缺字段 | 对照模板补全，缺失值填 `""` 或 `[]` |
| 章节缺失 | 对照模板补全缺失章节，内容留空 |

## 批量处理

1. 读取所有文件，按 `type` 分组
2. 每组对照对应模板，并行处理
3. 统一 frontmatter 字段值（枚举值对齐模板）
4. 验证 wikilink 引用有效性

## 质量标准

- [ ] `type` 字段存在且值合法（对照模板表）
- [ ] frontmatter 字段与对应模板一致，无多余/缺失字段
- [ ] 有且仅有一个 H1 标题
- [ ] 章节结构符合对应模板定义
- [ ] 所有代码块有语言标记
- [ ] 无外部死链图片
- [ ] 无孤立 `image.png` 行
- [ ] 中英文间有空格

## 相关技能

- [[knowledge-distillation]] — 拆分大型笔记
- [[meeting-summary]] — 格式化会议纪要
