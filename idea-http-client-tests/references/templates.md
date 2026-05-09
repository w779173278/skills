# IDEA HTTP Client Templates

## 1. 推荐目录模板

```text
src/test/resources/http/
├── http-client.env.json
├── http-client.private.env.json
├── README.md
└── {domain}/
    ├── README.md
    └── {bounded-context}/
        ├── README.md
        ├── {feature}.http
        ├── fragments/
        │   └── {shared-script}.js
        ├── request-body/
        │   └── {payload}.json
        └── output/
            └── .gitkeep
```

适用建议：
- `http-client.env.json`：公共、可提交、非敏感变量。
- `http-client.private.env.json`：密码、密钥、私有 token、cookie、证书引用；默认建议加入 `.gitignore`。
- `fragments/`：被 `import` 的脚本或共用片段。
- `request-body/`：复杂 JSON、multipart 文件。
- `output/`：导出响应文件时使用。

## 2. 公共环境文件模板

```json
{
  "local": {
    "host": "http://localhost:8080",
    "tenantCode": "GTZQ",
    "projectCode": "P1001",
    "operator": "devops"
  },
  "sit": {
    "host": "http://sit.example.com",
    "tenantCode": "GTZQ",
    "projectCode": "P1001",
    "operator": "sit-user"
  }
}
```

## 3. 私有环境文件模板

```json
{
  "local": {
    "username": "admin",
    "password": "change-me",
    "clientSecret": "change-me-too"
  },
  "sit": {
    "username": "sit-user",
    "password": "change-me",
    "clientSecret": "change-me-too"
  }
}
```

## 4. 根目录 README 模板

```md
# HTTP Client 测试资产

## 目录说明

- `http-client.env.json`：公共环境变量
- `http-client.private.env.json`：私有环境变量
- `{domain}/`：按业务域拆分的接口测试

## 使用方式

1. 在 IDEA 中打开 `.http` 文件并选择环境。
2. 先执行登录/初始化请求，再执行业务请求。
3. 批量回归时使用 `ijhttp` 或 Docker 版 HTTP Client CLI。

## 注意事项

- 私密变量不要写入公共环境文件。
- 需要归档测试结果时，使用 `--report`。
```

## 5. 领域目录 README 模板

```md
# 发布域 / 数据库发布

## 业务背景

本目录覆盖数据库+配置项发布相关接口，按限界上下文继续拆分。

## 调用顺序

1. 登录并获取 token
2. 查询或创建业务实体
3. 执行状态流转
4. 校验关键结果

## 关键环境变量

- `host`
- `tenantCode`
- `projectCode`
- `username`
- `password`
```

## 6. REST `.http` 模板

```http
# Feature: F01 交付能力配置
# Controller: DeliveryCapabilityController
# Prefix: /api/devops/release/capability
# Required env: host, tenantCode, operator

@scopeCode = PROJECT_1001

# @name queryCapabilityList
### [queryCapabilityList] 查询交付能力开关列表
GET {{host}}/api/devops/release/capability/list?scopeCode={{scopeCode}} HTTP/1.1
Accept: application/json
X-Operator: {{operator}}

> {%
  client.test("状态码应为 200", () => {
    client.assert(response.status === 200, "状态码不是 200");
  });
%}

###
```

## 7. 登录链路模板

```http
# @name login
### [login] 登录并保存 token
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

# @name queryAfterLogin
### [queryAfterLogin] 登录后查询
GET {{host}}/api/devops/release/capability/list?scopeCode={{scopeCode}}
Authorization: Bearer {{authToken}}
Accept: application/json

###
```

## 8. 预请求脚本模板

```http
< {%
  request.variables.set("requestId", $uuid);
  request.variables.set("timestamp", $isoTimestamp);
%}
POST {{host}}/api/devops/release/plans
Content-Type: application/json
X-Request-Id: {{requestId}}

{
  "createdAt": "{{timestamp}}"
}
```

## 9. OAuth2 环境模板

仅在 IDEA 内运行时使用；CLI 当前不支持 OAuth2。

```json
{
  "local": {
    "host": "https://example.org",
    "auth-dev": {
      "Type": "OAuth2",
      "Grant Type": "Client Credentials",
      "Token URL": "https://example.org/oauth/token",
      "Client ID": "http-client",
      "Client Secret": "{{clientSecret}}"
    }
  }
}
```

```http
GET {{host}}/api/secure/data
Authorization: Bearer {{$auth.token("auth-dev")}}
Accept: application/json
```

## 10. GraphQL 模板

```http
GRAPHQL {{host}}/graphql
Authorization: Bearer {{authToken}}

query queryReleasePlan($sheetId: Long!) {
  releasePlan(sheetId: $sheetId) {
    id
    stage
    status
  }
}

{
  "sheetId": 10001
}
```

## 11. WebSocket 模板

```http
WEBSOCKET {{wsHost}}/ws/release-events
Content-Type: application/json

===
{
  "event": "SUBSCRIBE",
  "sheetId": 10001
}
=== wait-for-server
```

## 12. gRPC 模板

```http
GRPC {{grpcHost}}/release.ReleaseQueryService/GetReleasePlan
Authorization: Bearer {{authToken}}
X-Tenant-Code: {{tenantCode}}

{
  "sheetId": "10001"
}
```
