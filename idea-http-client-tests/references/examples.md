# IDEA HTTP Client Examples

## 1. GET + 环境变量

```http
### [capabilityList] 查询交付能力列表
GET {{host}}/api/devops/release/capability/list?scopeCode={{projectCode}}
Accept: application/json
```

## 2. POST + `application/x-www-form-urlencoded`

```http
### [enableCapability] 启用交付能力
POST {{host}}/api/devops/release/capability/enable
Content-Type: application/x-www-form-urlencoded

scopeCode={{projectCode}}&capabilityCode=DATABASE_RELEASE
```

## 3. POST + `@RequestBody`

```http
### [createPlan] 创建交付计划
POST {{host}}/api/devops/release/plans
Content-Type: application/json

{
  "sheetId": 10001,
  "serviceId": 20001,
  "stage": "SIT"
}
```

## 4. DELETE

```http
### [deleteConfigBinding] 删除配置项绑定
DELETE {{host}}/api/devops/release/config-bindings/10001
Authorization: Bearer {{authToken}}
```

## 5. multipart/form-data

```http
### [importSqlBindings] 导入 SQL 绑定
POST {{host}}/api/devops/release/sql-bindings/import
Content-Type: multipart/form-data; boundary=ReleaseBoundary

--ReleaseBoundary
Content-Disposition: form-data; name="sheetId"
Content-Type: text/plain

10001
--ReleaseBoundary
Content-Disposition: form-data; name="file"; filename="sql-bindings.json"
Content-Type: application/json

< ./request-body/sql-bindings.json
--ReleaseBoundary--
```

## 6. 就地变量

```http
@basePath = /api/devops/release
@sheetId = 10001

### [querySheet] 查询提测单
GET {{host}}{{basePath}}/test-sheets/{{sheetId}}
Accept: application/json
```

## 7. 预请求变量

```http
< {%
  request.variables.set("requestId", $uuid);
  request.variables.set("now", $isoTimestamp);
%}
POST {{host}}/api/devops/release/freeze
Content-Type: application/json
X-Request-Id: {{requestId}}

{
  "sheetId": 10001,
  "freezeTime": "{{now}}"
}
```

## 8. 登录后提取 token

```http
### [login]
POST {{host}}/api/auth/login
Content-Type: application/json

{
  "username": "{{username}}",
  "password": "{{password}}"
}

> {%
  client.test("登录成功", () => {
    client.assert(response.status === 200, "登录失败");
    client.assert(response.body.token, "token 缺失");
  });
  client.global.set("authToken", response.body.token);
%}

###

### [queryCurrentUser]
GET {{host}}/api/auth/current-user
Authorization: Bearer {{authToken}}
Accept: application/json
```

## 9. JSONPath 断言

```http
### [queryFreezeResult]
GET {{host}}/api/devops/release/plans/10001
Accept: application/json

> {%
  client.test("状态应为 FROZEN", () => {
    const status = jsonPath(response.body, "$.data.status");
    client.assert(status === "FROZEN", "状态不是 FROZEN");
  });
%}
```

## 10. 循环集合变量

```http
< {%
  request.variables.set("services", [
    {"id": 20001, "stage": "DEV"},
    {"id": 20002, "stage": "SIT"}
  ]);
%}
POST {{host}}/api/devops/release/plan/query
Content-Type: application/json

{
  "serviceId": {{$.services..id}},
  "stage": "{{$.services..stage}}"
}

> {%
  // request.iteration() 返回当前循环索引（第一项为 0）
  const current = request.variables.get("services")[request.iteration()];
  client.test(`服务 ${current.id} 查询成功`, () => {
    client.assert(response.status === 200, "查询失败");
  });
%}
```

## 10b. request.templateValue() — 按位置取当次迭代的模板值

当请求体包含多个 JSONPath 集合变量时，响应脚本中用 `request.templateValue(index)` 按参数位置（0-based）获取当次迭代的值：

```http
< {%
  request.variables.set("plans", [
    {"planId": 10001, "stage": "SIT"},
    {"planId": 10002, "stage": "UAT"},
    {"planId": 10003, "stage": "PRD"}
  ]);
%}
POST {{host}}/api/devops/release/plan/submit
Content-Type: application/json

{
  "planId": {{$.plans..planId}},
  "stage":  "{{$.plans..stage}}"
}

> {%
  // templateValue(0) → 第一个模板参数（planId）当前迭代值
  // templateValue(1) → 第二个模板参数（stage）当前迭代值
  const planId = request.templateValue(0);
  const stage  = request.templateValue(1);
  client.test(`planId=${planId} stage=${stage} 提交成功`, () => {
    client.assert(response.status === 200, "提交失败");
  });
%}
```

## 11. 引入共享脚本

```http
< {%
  import {makeSignature} from "./fragments/signature";
  request.variables.set("signature", makeSignature());
%}
POST {{host}}/api/devops/release/callback
Content-Type: application/json
X-Signature: {{signature}}

{
  "event": "PACKAGED"
}
```

## 12. 运行其他请求文件

```http
import common/auth.http
import plan/plan-query.http

run common/auth.http
run #queryFreezeResult
run #queryFreezeResult (@sheetId = 10002)
```

## 13. 输出响应到文件

```http
GET {{host}}/api/devops/release/report/export >> ./output/release-report.json
GET {{host}}/api/devops/release/report/export >>! ./output/release-report.json
```

## 18. 请求控制标志

```http
# @no-redirect
### [testRedirect] 测试重定向行为（不跟随，直接返回 3xx）
GET {{host}}/api/devops/release/redirect

###

# @no-log
### [healthCheck] 健康检查（排除历史记录）
GET {{host}}/api/devops/release/health

###

# @timeout 30 s
# @connection-timeout 5 s
### [slowExport] 慢接口导出（自定义超时）
GET {{host}}/api/devops/release/slow-export
```

## 19. 读取系统环境变量

适用于 CI 场景：密码、token 直接来自 CI 环境变量，不写入任何文件。

```http
### [loginWithEnvSecret] 用系统环境变量传密码
POST {{host}}/api/auth/login
Content-Type: application/json

{
  "username": "{{$env.RELEASE_USERNAME}}",
  "password": "{{$env.RELEASE_PASSWORD}}"
}
```

也可在 CLI 中通过 `-P` 注入：

```bash
ijhttp --env-file http-client.env.json --env dev \
  -P "RELEASE_PASSWORD=$MY_SECRET" \
  release/auth.http
```

## 20. 响应头提取

```http
### [createResource] 创建资源并提取 Location 头
POST {{host}}/api/devops/release/plans
Content-Type: application/json

{
  "sheetId": 10001,
  "stage": "SIT"
}

> {%
  client.test("创建成功", function() {
    client.assert(response.status === 201, "期望 201");
  });

  // 提取 Location 头（单个值）
  const location = response.headers.valueOf("Location");
  if (location) {
    client.global.set("flow_plan_location", location);
  }

  // 提取多个同名头（如 Set-Cookie）
  const cookies = response.headers.valuesOf("Set-Cookie");
  if (cookies.length > 0) {
    client.global.set("flow_session_cookie", cookies[0]);
  }
%}
```

## 22. 动态变量本地化（2026.1 新增）

生成符合特定地区格式的随机数据，避免测试数据全是英文：

```http
### [createLocalizedTestData] 用中文格式随机数据创建测试用户
@testName    = {{$random.locale.name.fullName("zh_CN")}}
@testPhone   = {{$random.locale.phoneNumber.cellPhone("zh_CN")}}
@testAddress = {{$random.locale.address.fullAddress("zh_CN")}}

POST {{host}}/api/devops/release/test-users
Content-Type: application/json

{
  "name":    "{{testName}}",
  "phone":   "{{testPhone}}",
  "address": "{{testAddress}}"
}
```

如需幂等性（相同 seed 每次生成完全一致的值）：

```http
@testName = {{$random.locale.name.fullName("zh_CN", 42)}}
```

在预请求脚本中读取文件内变量：

```http
< {%
  const name = client.variables.file.get("testName");
  client.log("当前测试姓名：" + name);
%}
```

## 21. 事件流处理（SSE / NDJSON）

```http
### [watchReleaseEvents] 监听发布事件流
GET {{host}}/api/devops/release/plans/10001/events
Accept: text/event-stream

> {%
  response.body.onEachLine((data, unsubscribe) => {
    if (!data || data.startsWith(":")) return; // 跳过心跳行

    const line = data.startsWith("data: ") ? data.slice(6) : data;
    let event;
    try {
      event = JSON.parse(line);
    } catch (e) {
      return;
    }

    client.test("事件有 status 字段", function() {
      client.assert(event.status !== undefined, "status 缺失");
    });

    if (event.status === "DONE") {
      unsubscribe();
    }
  });
%}
```

## 14. GraphQL

```http
GRAPHQL {{host}}/graphql
Authorization: Bearer {{authToken}}

query queryRelease($sheetId: Long!) {
  releasePlan(sheetId: $sheetId) {
    id
    stage
    status
  }
}

{
  "sheetId": 10001
}

> {%
  client.test("GraphQL 查询成功", () => {
    client.assert(response.body.data.releasePlan.status !== undefined, "status 缺失");
  });
%}
```

## 15. WebSocket

```http
WEBSOCKET {{wsHost}}/ws/release-events
Content-Type: application/json

===
{"event":"SUBSCRIBE","sheetId":10001}
=== wait-for-server

> {%
  let seen = 0;
  response.body.onEachMessage((message, unsubscribe) => {
    seen++;
    const payload = JSON.parse(message);
    client.test("收到状态事件", () => {
      client.assert(payload.status !== undefined, "status 缺失");
    });
    if (seen === 1) {
      unsubscribe();
    }
  });
%}
```

## 16. gRPC

```http
GRPC grpcs://{{grpcHost}}/release.ReleaseQueryService/GetReleasePlan
Authorization: Bearer {{authToken}}

{
  "sheetId": "10001"
}
```

说明：
- 需要 Protocol Buffers 与 gRPC 插件。
- 具体支持能力以当前 IDE 帮助与已安装插件为准。

## 17. CLI 命令

### `ijhttp` — 基本运行

```bash
ijhttp \
  --env-file src/test/resources/http/http-client.env.json \
  --private-env-file src/test/resources/http/http-client.private.env.json \
  --env local \
  src/test/resources/http/release/api/capability.http \
  --report
```

### `ijhttp` — 命令行覆盖变量（无需修改 env 文件）

```bash
# -V 覆盖公共变量，-P 覆盖私有变量（可重复）
ijhttp \
  --env-file http-client.env.json --env dev \
  -V host=http://localhost:8080 \
  -V projectCode=TEST001 \
  -P authToken=my-secret-token \
  release/api/capability.http
```

### `ijhttp` — 详细日志

```bash
# -L HEADERS 打印请求/响应头，-L VERBOSE 额外打印 Body
ijhttp -L VERBOSE --env-file http-client.env.json --env dev release/api/capability.http
```

### Docker

```bash
docker run --rm -i -t \
  -v "$PWD:/workdir" \
  jetbrains/intellij-http-client \
  --env-file src/test/resources/http/http-client.env.json \
  --private-env-file src/test/resources/http/http-client.private.env.json \
  --env local \
  src/test/resources/http/release/api/capability.http \
  --report
```

### Docker 访问宿主机 localhost

```bash
docker run --rm -i -t \
  -v "$PWD:/workdir" \
  --add-host host.docker.internal:host-gateway \
  jetbrains/intellij-http-client \
  -D \
  --env-file src/test/resources/http/http-client.env.json \
  --env local \
  src/test/resources/http/release/api/capability.http
```

### Docker — CI 命令行注入私有变量

```bash
docker run --rm -i -t \
  -v "$PWD:/workdir" \
  jetbrains/intellij-http-client \
  --env-file src/test/resources/http/http-client.env.json \
  --env ci \
  -P "authToken=$CI_AUTH_TOKEN" \
  -P "dbPassword=$CI_DB_PASSWORD" \
  src/test/resources/http/release/api/capability.http \
  --report
```

---

## 官方 HTTP Requests Collection 参考

IntelliJ IDEA 自带的 HTTP Client 示例已完整拷贝至 [collection/](collection/) 目录，可作为语法参考：

| 文件 | 覆盖内容 |
|------|----------|
| [get-requests.http](collection/get-requests.http) | GET 请求、环境变量、动态变量、Cookie、HTTP/2、响应保存 |
| [post-requests.http](collection/post-requests.http) | POST JSON、form-urlencoded、multipart/form-data、动态变量 |
| [requests-with-tests.http](collection/requests-with-tests.http) | 响应断言、crypto hash/HMAC、预请求脚本、import 共享脚本、global 变量 |
| [requests-with-authorization.http](collection/requests-with-authorization.http) | Basic/Digest/Bearer Token 认证、token 链式传递 |
| [requests-with-loop.http](collection/requests-with-loop.http) | 循环遍历集合变量、JSONPath 断言、`request.iteration()` |
| [requests-with-include.http](collection/requests-with-include.http) | `import`/`run` 引入与执行其他 `.http` 文件、参数覆盖 |
| [graphql-requests.http](collection/graphql-requests.http) | GraphQL HTTP/WebSocket 查询、mutation、subscription、AWS AppSync |
| [ws-requests.http](collection/ws-requests.http) | WebSocket 连接、消息分隔、`onEachMessage` 脚本 |
| [different-responses.http](collection/different-responses.http) | 图片、PDF、HTML 预览响应 |
| [my-utils.js](collection/my-utils.js) | 可导入的共享脚本模块示例 |
| [environment/](collection/environment/) | `http-client.env.json` 和 `http-client.private.env.json` 环境文件示例 |
