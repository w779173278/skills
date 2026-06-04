---
name: source-code-analysis
description: 分析 Java 框架源码，产出结构化学习笔记。追踪执行流程，识别关键类/接口，提取设计模式，生成带代码引用的文档。适用场景：分析 Spring/MyBatis/Netty 源码、创建源码学习笔记，或用户提及"源码分析"、"源码学习"、"源码解读"。
priority: P1
origin: user
license: MIT
---

# Source Code Analysis

分析 Java 框架源码，产出结构化学习笔记。

## 分析流程

### 1. 环境准备

- 确认源码版本（与项目使用版本一致）
- 搭建可调试的源码阅读环境
- 记录 Maven/Gradle 依赖坐标

### 2. 入口定位

| 框架 | 典型入口 |
|------|---------|
| Spring IOC | `AbstractApplicationContext.refresh()` |
| Spring AOP | `ProxyFactory.getProxy()` |
| Spring MVC | `DispatcherServlet.doDispatch()` |
| MyBatis | `SqlSessionFactoryBuilder.build()` |
| Netty | `ServerBootstrap.bind()` |

### 3. 执行流追踪

```
入口方法
  → 核心流程方法
    → 关键子流程
      → 具体实现
```

每个层级记录：
- 类名 + 方法签名
- 核心逻辑（3-5 行关键代码）
- 设计模式识别
- 扩展点 / SPI

### 4. 笔记产出格式

使用 [[system]] 模板，`doc_type: explanation`。EKWS 映射：

| EKWS 层 | system 模板章节 | 源码分析内容 |
|---------|---------------|------------|
| Problem | 解决什么问题 | 为什么分析这段源码 |
| Flow | 整体流程 | 执行链路追踪 |
| Component | 核心模块 | 关键类/接口表 |
| Detail | 关键实现 | 精简源码片段 + 设计模式 |

额外在「关键实现」后补充设计模式识别（在 `## 关键实现` 内部，不新增章节）：

```markdown
## 设计模式

- **工厂模式**: 用于...
- **代理模式**: 用于...
```

### 5. 精简原则

- 代码片段：只保留核心逻辑行，删除日志/异常处理/参数校验
- 大段代码用 `// ...省略...` 标记
- 每个源码分析笔记控制在 300-400 行
- 超过 500 行必须拆分

## 常用调试技巧

```java
// 在关键方法打断点，观察调用栈
// 使用 Evaluate Expression 查看运行时变量
// 条件断点：右键断点 → Condition
```

## 相关技能

- [[knowledge-distillation]] — 拆分大型源码笔记
- [[document-polish]] — 格式化输出笔记
