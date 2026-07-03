# API 契约与错误码定义

## 1. 核心 API 契约与传输格式
[此处写明传输协议如 gRPC/HTTP，并提供主要传输实体格式]

```protobuf
syntax = "proto3";
package com.example.api.v1;

// 服务请求结构
message ExecuteRequest {
    string request_id = 1;
    string key = 2;
    bytes payload = 3;
    map<string, string> attributes = 4;
}

// 服务应答结构
message ExecuteResponse {
    int32 code = 1;
    string message = 2;
    string request_id = 3;
}
```

## 2. 状态码与异常对照字典
提供运行时抛出的常见业务或系统错误对照：

| 响应状态码 (Code) | HTTP/gRPC 对照码 | 场景描述 | 客户端处理建议 |
| :--- | :--- | :--- | :--- |
| `ENTITY_NOT_FOUND` | NOT_FOUND (5) | 服务端没有查找到指定的实体资源。 | 检查传入的资源 Key 是否存在或拼写是否正确。 |
| `RATE_LIMIT_EXCEEDED` | RESOURCE_EXHAUSTED (8) | 触发了服务端接口流量限制（限流）。 | 客户端触发指数退避重试 (Exponential Backoff)。 |
| `INVALID_PARAMETER` | INVALID_ARGUMENT (3)| 请求报文校验失败，包含非法参数或格式错误。 | 检查输入参数格式。 |
