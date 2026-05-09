---
name: idea-http-client-tests
description: Use when generating, reorganizing, or maintaining IntelliJ IDEA HTTP Client `.http` / `.rest` test assets for a Java codebase, especially when REST, GraphQL, WebSocket, or gRPC debug requests need to become portable repository files with environment files, README guidance, reusable assertions, and `ijhttp`/Docker execution instructions.
---

# IDEA HTTP Client Tests

## Goal

把零散接口说明、调试请求和联调脚本整理成可提交 Git、可在 IDEA 和 `ijhttp`/Docker 中复用的 HTTP Client 测试资产。

默认目标目录仍是 `src/test/resources/http/`，但要优先复用仓库已有目录约定。

## Workflow

1. 先识别边界与运行方式。
   - 查 Controller、请求 DTO、统一前缀、鉴权方式、环境切换方式、是否已有 `.http` 资产。
   - 判断这是 REST、GraphQL、WebSocket 还是 gRPC；不要把不同协议混在一个“万能模板”里。
   - 若仓库是 DDD / 多模块，先按领域、限界上下文、feature 规划目录，再落文件。

2. 先搭骨架，再写请求。
   - 默认准备公共环境文件 `http-client.env.json`。
   - 需要密码、密钥、cookie、私有 token 时，再补 `http-client.private.env.json`。
   - 为领域目录或 feature 目录补 `README.md`，写业务背景、调用顺序、环境变量、前置条件。

3. 按真实 Spring MVC / OpenAPI / proto 语义还原请求。
   - `GET` 参数放 URL。
   - `POST` + `@RequestParam` 优先 `application/x-www-form-urlencoded`。
   - `POST`/`PUT` + `@RequestBody` 用 JSON。
   - `multipart/form-data`、GraphQL、WebSocket、gRPC 直接用 HTTP Client 原生语法，不要退化成伪示例。
   - 字段名、Header 名、枚举值、路径必须来自代码、文档或 `.proto`，不要脑补。

4. 只写有价值的脚本与断言。
   - 链式流程用 `client.global`、`request.variables` 或 `client.variables` 传递值。
   - 对稳定字段写断言：状态码、`success`、关键 ID、token、集合长度、JSONPath 结果。
   - 不对易变字段写脆弱断言：整段 message、完整时间戳、排序不稳定列表。

5. 让资产可迁移、可批量执行。
   - 优先保证在 IDEA 中直接运行。
   - 命令行执行优先 `ijhttp`，其次官方 Docker 镜像 `jetbrains/intellij-http-client`。
   - 需要 CI 归档时，提供 `--report` 方案。
   - 明确 IDE-only 能力与 CLI 限制，尤其 OAuth2 / SSL 配置不能假设 CLI 也支持。

## Authoring Rules

- 一个 `.http` 文件聚焦一个 feature 或一组强相关接口。
- 请求块之间用 `###` 分隔；需要复用或跳转时用 `# @name`。
- Headers 与 Body 之间必须留一个空行。
- 仓库中已有 `.http` 风格时，优先沿用；不要为了“统一”重排所有文件。
- 优先用环境变量、就地变量、预请求变量表达差异，不要复制 N 份仅 host 不同的请求。
- 对批量/循环请求，优先用 JSONPath 集合遍历，不要手工复制十几段请求。
- 如果用户只让你补单个接口，也要顺手检查环境文件、README、前置登录或公共脚本是否缺失。

## Protocol Boundaries

- REST：默认主路径，最常见。
- GraphQL：可直接用 `GRAPHQL` 请求，不要伪装成普通 `POST` JSON。
- WebSocket：可直接用 `WEBSOCKET`，支持消息分隔和脚本监听。
- gRPC：如项目已启用相关插件，可按 `GRPC` 语法组织请求；具体能力以当前 IDE 帮助与已安装插件为准。

## CLI Guidance

- 本机装了 `ijhttp` 时，优先给出 `ijhttp` 命令。
- 没装时优先给 Docker 版 `jetbrains/intellij-http-client`，并挂载到容器 `/workdir`。
- 目标服务跑在宿主机 `localhost` 时，Docker 命令加 `-D`；Linux Docker 20.10.0+ 再加 `--add-host host.docker.internal:host-gateway`。
- 需要 CI 测试报告时，用 `--report` 输出 JUnit XML。
- 注意：JetBrains 2026.1 文档明确说明 CLI 当前不支持 OAuth 2.0 和 SSL configuration；这些能力仅能在 IDE 侧完整使用。

## What To Read

- 需要目录骨架、环境文件、README、feature 文件模板时，读 [references/templates.md](references/templates.md)。
- 需要命名约定、变量规则、断言原则、CLI/IDE 边界时，读 [references/rules.md](references/rules.md)。
- 需要现成语法片段和多协议示例时，读 [references/examples.md](references/examples.md)。
- 需要查看 IntelliJ IDEA 官方 HTTP Client 原始示例时，浏览 [references/collection/](references/collection/) 目录。
- 需要总览入口或兼容旧引用时，读 [references/http-client-patterns.md](references/http-client-patterns.md)。

## Source Notes

- 本 skill 已参考 IntelliJ IDEA 自带 HTTP Requests Collection：
      `/Users/weishengdong/Applications/IntelliJ IDEA.app/Contents/plugins/restClient/lib/restClient/com/intellij/ws/rest/client/requests/collection`
- 也对照了 JetBrains 官方文档（2025.3 / 2026.1）：
  - HTTP Client: `https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html`
  - HTTP Syntax: `https://www.jetbrains.com.cn/help/idea/2025.3/exploring-http-syntax.html`
  - HTTP Client Variables (2025.3): `https://www.jetbrains.com.cn/help/idea/2025.3/http-client-variables.html`
  - HTTP Client Variables (2026.1, 含多环境文件管理 + 本地化动态变量): `https://www.jetbrains.com.cn/help/idea/2026.1/http-client-variables.html`
  - HTTP Response Handling Examples: `https://www.jetbrains.com.cn/help/idea/2025.3/http-response-handling-examples.html`
  - HTTP Client CLI: `https://www.jetbrains.com.cn/help/idea/2025.3/http-client-cli.html`
  - OAuth 2.0 authorization: `https://www.jetbrains.com/help/idea/oauth-2-0-authorization.html`
