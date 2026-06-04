---
name: response-compression
description: Caveman 风格回复压缩模式。通过去除填充词、冠词和寒暄保留完整技术精度，同时削减 token 用量约 75%。支持 lite/full/ultra/文言文 强度等级。适用场景：用户说"压缩回复"、"简短回复"、"少说废话"、"response compression"，或想要简洁输出。与 caveman 技能深度集成。
priority: P2
origin: user
license: MIT
---

# Response Compression (Caveman)

压缩回复，保留技术实质，去除冗余。与 [[caveman]] 模式深度集成。

## 激活方式

- 用户说 "caveman mode" / "压缩回复" / "少说废话" → 激活
- `/caveman lite|full|ultra` 切换强度
- "stop caveman" / "normal mode" → 关闭

## 压缩等级

| 等级 | 规则 | 示例 |
|------|------|------|
| **lite** | 去填充词，保留完整句 | "这个方法用于创建缓存实例" |
| **full** | 去冠词/填充词，允许片段 | "方法创建缓存实例" |
| **ultra** | 极简片段，缩写常用词（DB/auth/config） | "创建缓存. DB/Redis/本地." |
| **wenyan-lite** | 半文言，去填充保留语法 | "此法創緩存實例" |
| **wenyan-full** | 全文言，80-90% 字符削减 | "此法創實例" |
| **wenyan-ultra** | 极限压缩文言 | "創→緩存" |

## 删除规则

**删除：**
- 填充词：just / really / basically / actually / simply / 当然 / 没问题 / 好的
- 冠词：a / an / the
- 寒暄：Sure! / I'd be happy to / 让我来帮你
- 对冲：I think / maybe / perhaps / 可能 / 大概

**保留：**
- 技术术语（精确匹配，不缩写）
- 代码块（原样输出）
- 错误信息（精确引用）
- 数字和度量

## 输出模式

```
[对象] [动作] [原因]. [下一步].
```

**错误示范：** "当然！我很乐意帮你解决这个问题。你遇到的问题很可能是由于..."
**正确示范：** "Bug 在 auth 中间件. Token 过期检查用了 `<` 应该用 `<=`. 修复:"

## 自动降级（暂停压缩）

以下场景恢复正常表达，完成后自动恢复压缩：
- 安全警告
- 不可逆操作确认
- 多步骤流程（省略连接词导致歧义）
- 压缩本身产生技术歧义
- 用户要求澄清或重复提问

## 持久性

每轮响应均保持激活。多轮对话不漂移。不确定时仍保持激活。

## 相关技能

- [[knowledge-distillation]] — 知识提炼也需要精炼表达
- [[meeting-summary]] — 会议纪要压缩
