# API 契约与错误码定义

## 1. 核心 API 契约与传输格式
[此处写明传输协议如 gRPC/HTTP，并提供主要传输实体格式]

```protobuf
syntax = "proto3";
package apache.rocketmq.v2;

// 发送消息的请求结构
message SendMessageRequest {
    string message_id = 1;
    string topic = 2;
    bytes body = 3;
    map<string, string> user_properties = 4;
    int64 delivery_timestamp = 5; // 用于定时/延迟消息
}

// 发送消息的应答结构
message SendMessageResponse {
    Status status = 1;
    string message_id = 2;
    string transaction_id = 3;
}
```

## 2. 状态码与异常对照字典
提供运行时抛出的常见业务或系统错误对照：

| 响应状态码 (Code) | HTTP/gRPC 对照码 | 场景描述 | 客户端处理建议 |
| :--- | :--- | :--- | :--- |
| `TOPIC_NOT_FOUND` | NOT_FOUND (5) | 服务端没有查找到指定的 Topic 路由。 | 检查 TopicName 是否正确，或是否已在 NameServer 注册。 |
| `TOO_MANY_REQUESTS` | UNAVAILABLE (14) | 触发了服务端流控（限流）。 | 客户端触发指数退避重试 (Exponential Backoff)。 |
| `MESSAGE_CORRUPTED` | INVALID_ARGUMENT (3)| 消息体校验和校验失败，数据可能在传输中损坏。 | 重新生成消息并发送。 |
