---
name: incident-analysis
description: 结构化故障分析，产出复盘文档。遵循 5-Whys 方法，识别根本原因，生成有负责人和时间线的行动项。适用场景：分析生产故障、编写事故复盘，或用户提及"故障分析"、"事故复盘"、"incident"、"RCA"。
priority: P1
origin: user
license: MIT
---

# Incident Analysis

结构化故障分析，产出复盘文档。

## 分析框架

### 1. 时间线还原

```
HH:MM  触发条件/变更
HH:MM  监控告警触发
HH:MM  值班人员介入
HH:MM  定位根因
HH:MM  执行修复
HH:MM  服务恢复
```

### 2. 5-Whys 分析

从表象逐层追问：
```
现象: API 响应超时
Why 1: 数据库连接池耗尽
Why 2: 慢查询未释放连接
Why 3: 新上线的报表接口缺少索引
Why 4: 上线前未做 SQL Review
Why 5: 缺少 Code Review 中的 SQL 检查项
→ 根因: 流程缺陷
```

### 3. 影响评估

| 维度 | 评估 |
|------|------|
| 影响范围 | 用户数/请求数/地域 |
| 持续时间 | 分钟级/小时级 |
| 业务损失 | 订单/营收/SLA |
| 数据完整性 | 是否有数据丢失/不一致 |

### 4. 复盘文档格式

使用 [[incident]] 模板。`severity` 字段填 p0/p1/p2/p3，`status` 填 `postmortem`。

5-Whys 分析写入 `root-causes` 章节，每层 Why 作为列表项。

## 分析原则

- **对事不对人** — 聚焦系统和流程，而非个人失误
- **数据驱动** — 用监控数据/日志还原事实
- **可操作** — 每个改进项有负责人和截止日期
- **透明** — 全员可见，促进学习文化

## 相关技能

- [[architecture-review]] — 从架构角度评估故障根因
- [[document-polish]] — 格式化复盘文档
