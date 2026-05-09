# IDEA HTTP Client Rules

## 1. 目录与拆分规则

- 默认根目录为 `src/test/resources/http/`。
- 优先按业务域 / 限界上下文 / feature 拆分，不按“张三调试”“临时联调”之类个人维度拆分。
- 一个 `.http` 文件只承载一个 feature 或一个强相关链路。
- 多场景接口要拆成多个请求块，不写“万能参数大杂烩”。
- 仓库已有约定时，优先复用现有目录与命名。

## 2. 命名规则

- 文件名推荐 `Sxx_action_subject.http` 或 `{feature}.http`，与仓库现状保持一致。
- 关键请求加 `# @name`，名称用英文驼峰或蛇形，保持稳定可搜索。
- `README.md` 用中文写业务语义；接口路径、Header、字段、枚举保持代码原文。

## 3. 请求编写规则

- 请求块之间用 `###` 分隔。
- 请求头和请求体之间必须有一个空行。
- 注释要解释业务动机、触发时机、调用后效果，不要只重复 URL。
- 能从 Controller / DTO / OpenAPI / proto 得出的内容，不再凭经验脑补。
- `application/x-www-form-urlencoded` 场景注意 `%` 转义规则；不要手写错误的 `&`、`=`、`+`。

## 3.1 请求控制标志（Request Flags）

在请求块前的注释行加 `# @flag` 可控制该请求的运行行为，常用标志：

| 标志 | 作用 |
|------|------|
| `# @no-redirect` | 禁止跟随 3xx 重定向（测试重定向接口本身时有用） |
| `# @no-log` | 排除出请求历史记录（高频健康检查避免日志污染） |
| `# @no-cookie-jar` | 禁止本次请求使用 Cookie 存储 |
| `# @no-auto-encoding` | 禁用响应体自动解码 |
| `# @timeout 600` | 设置数据包超时，支持 `ms`、`s`、`m` 单位 |
| `# @connection-timeout 2 m` | 设置连接建立超时 |

示例：
```http
# @no-redirect
# @timeout 30 s
GET {{host}}/api/devops/release/redirect-test
```

## 4. 变量规则

### 变量优先级（高 → 低）

| 优先级 | 类型 | 定义方式 |
|--------|------|---------|
| 1（最高） | 环境变量 | `http-client.env.json` / `http-client.private.env.json` |
| 2 | 全局变量 | `client.global.set(...)` |
| 3 | 文件内变量（In-place） | `@var = value`（作用域：整个 `.http` 文件） |
| 4 | 单请求变量（Per-request） | `request.variables.set(...)`（作用域：紧随的单个请求） |
| 5（最低） | 内置动态变量 | `{{$uuid}}`、`{{$timestamp}}` 等保留名称 |

统一访问入口：`client.variables`（可读取所有作用域变量）。

### 约束

- 公共文件只放 `host`、端口、默认业务参数、非敏感 token。
- 私有文件放密码、私钥、client secret、cookie 等敏感信息；私有文件必须加入 `.gitignore`。
- 同名变量时，私有环境文件覆盖公共环境文件同名 key。
- 存在多层目录时，HTTP Client 优先解析当前目录的环境文件，找不到再向上递归查父目录；设计目录时要避免同名环境造成误判。

### 多目录多环境文件管理

当项目有多个服务子目录，每个子目录需要不同的私有变量（如不同的 API key）时：

```
root/
  http-client.env.json           # 公共，含 host 等通用变量
  service1/
    http-client.private.env.json # 私有，service1 专属 key
  service2/
    http-client.private.env.json # 私有，service2 专属 key
```

- service1 目录下的 `.http` 文件运行时，取 service1 的私有变量。
- service2 同理，互不干扰。
- IDEA 环境选择器分"For File"（当前目录及父目录）和"From Whole Project"（其他位置）两个区域，同名环境只显示一次。

### 含点号的变量名

变量名含点号时必须用方括号访问，否则会被解析为 JSONPath：

```http
GET {{client.['host.url']}}/api/v1/users
```

## 5. 动态变量与 JSONPath 规则

- 2024.2 起，除内置动态变量外，HTTP Client 将普通变量视作 JSONPath 表达式。
- 变量名中如果使用点号，必须保证对应值是 JSON 结构，而不是扁平字符串。
- 批量请求优先使用集合变量 + JSONPath，而不是复制多份请求。
- 兼容旧写法与新写法：
  - 兼容：`{{$uuid}}`、`{{$randomInt}}`
  - 新写法：`{{$random.uuid}}`、`{{$random.integer(10, 100)}}`
- 需要在脚本里访问动态变量时，直接使用 JavaScript 变量形式，例如 `$uuid`、`$isoTimestamp`。

### 完整动态变量列表

| 变量 | 说明 |
|------|------|
| `{{$uuid}}` / `{{$random.uuid}}` | UUID v4 |
| `{{$timestamp}}` | Unix 时间戳（秒） |
| `{{$isoTimestamp}}` | ISO-8601 格式时间 |
| `{{$randomInt}}` / `{{$random.integer(0, 100)}}` | 随机整数，新写法支持范围 |
| `{{$random.float(1.0, 9.9)}}` | 指定范围浮点数 |
| `{{$random.alphabetic(8)}}` | 纯字母随机串 |
| `{{$random.alphanumeric(12)}}` | 字母数字混合随机串 |
| `{{$random.hexadecimal(16)}}` | 十六进制随机串 |
| `{{$random.email}}` | 随机邮箱地址 |
| `{{$exampleServer}}` | HTTP Client 内置测试服务器地址 |
| `{{$env.VAR_NAME}}` | 读取系统环境变量（不依赖 env.json） |

`{{$env.VAR_NAME}}` 适合 CI 场景：密码、密钥直接从 CI 环境变量注入，不写入任何文件。

`$random` 还支持更多 Java Faker 类，输入 `{{$random.` 即可在 IDEA 中触发补全，常用类包括：`address`、`name`、`company`、`internet`、`lorem`、`phoneNumber`、`color`、`finance`、`hacker` 等。

### 动态变量本地化（2026.1 新增）

`$random.locale.*` 支持生成特定地区格式的随机数据，避免全英文假数据与本地业务不符：

```http
@testName    = {{$random.locale.name.fullName("zh_CN")}}
@testPhone   = {{$random.locale.phoneNumber.cellPhone("zh_CN")}}
```

- 第一个参数为语言/地区代码（如 `"zh_CN"`、`"fr"`、`"ja"`）。
- 第二个参数（可选）为 seed：相同 locale + 相同 seed 每次生成完全一致的值，适合需要幂等性的测试场景。

```http
@testName = {{$random.locale.name.fullName("zh_CN", 42)}}
```

## 6. 断言规则

推荐断言：
- `response.status`
- `response.contentType.mimeType`
- `response.headers.valueOf("Header-Name")` — 提取单个 Header 值
- `response.headers.valuesOf("Set-Cookie")` — 提取多个同名 Header 值（返回数组）
- `response.body` 中稳定字段
- `jsonPath(response.body, "...")` 的稳定结果

避免断言：
- 完整 message 文案
- 时间戳全文匹配
- 顺序不稳定的大列表全文
- 含随机值的整个响应体

### 推荐断言模式（完整）

```javascript
> {%
  client.test("状态码 200", function() {
    client.assert(response.status === 200, "期望 200，实际 " + response.status);
  });

  client.test("Content-Type 应为 JSON", function() {
    client.assert(
      response.contentType.mimeType === "application/json",
      "期望 JSON，实际 " + response.contentType.mimeType
    );
  });

  client.test("data 不为空", function() {
    client.assert(response.body.data != null, "data 字段缺失");
  });

  // 提取 ID 供后续请求使用
  const id = response.body.data && response.body.data.id;
  if (id) {
    client.global.set("flow_entity_id", String(id));
  }
%}
```

## 7. 链式调用规则

- 登录、换票据、创建实体再查询这类链路，优先使用 `client.global.set(...)`。
- 仅当前请求使用的数据，优先 `request.variables.set(...)`。
- 复杂前置脚本可拆到 `fragments/*.js` 并通过 `import` 复用。
- 用 `run` / `import` 串联已有请求时，避免制造隐式依赖过深的“黑盒大脚本”。

## 8. 协议选择规则

- REST：默认选项。
- GraphQL：用 `GRAPHQL` 关键字，变量 JSON 独立成块。
- WebSocket：用 `WEBSOCKET`，消息用 `===` 分隔。
- gRPC：如项目已启用相关插件，可按 `GRPC` 语法组织请求；具体能力以当前 IDE 帮助与已安装插件为准。

## 9. IDE 与 CLI 边界

IDE 强项：
- OAuth 2.0 配置与 `$auth.token(...)`
- SSL/TLS 与证书相关配置
- OpenAPI / proto 辅助生成、补全、导航

CLI 规则：
- 优先 `ijhttp`；未安装时优先 Docker 镜像 `jetbrains/intellij-http-client`
- Docker 执行必须挂载到 `/workdir`
- 访问宿主机 `localhost` 时加 `-D`
- Linux Docker 20.10.0+ 再加 `--add-host host.docker.internal:host-gateway`
- CI 回归默认建议加 `--report`

### CLI 完整参数表

| 参数 | 说明 |
|------|------|
| `--env-file <file>` | 指定公共环境变量文件 |
| `--private-env-file <file>` | 指定私有环境变量文件 |
| `--env <name>` | 选择环境名称（如 `dev`、`test`） |
| `-V key=value` | 命令行覆盖公共变量（可重复使用） |
| `-P key=value` | 命令行覆盖私有变量（可重复使用） |
| `-L HEADERS` | 日志输出请求和响应头 |
| `-L VERBOSE` | 日志输出请求和响应头 + Body |
| `-D` | 将 `localhost` 解析到宿主机（Docker 内使用） |
| `--report` | 输出 JUnit XML 报告（`report.xml`） |
| `--version` | 查看版本 |

### CLI 安装方式

```bash
# macOS Homebrew
brew install ijhttp

# Docker（CI 推荐，无需本地安装）
docker pull jetbrains/intellij-http-client
```

CLI 限制：
- JetBrains 2025.3 文档明确：HTTP Client CLI 当前不支持 OAuth 2.0 authorization
- JetBrains 2025.3 文档明确：HTTP Client CLI 当前不支持 SSL configuration

## 10. 推荐输出内容

执行这类任务时，优先给出：
- 目录结构
- 新增/更新的 `.http` 文件
- 环境文件变更
- README 说明
- 在 IDEA 中如何运行
- 在 `ijhttp` / Docker 中如何批量运行
- 是否需要 `.gitignore` 补私有环境文件
