---
type: docs
title: "Kafka 绑定规范"
linkTitle: "Kafka"
description: "Kafka 组件绑定详细说明"
aliases:
  - "/zh-hans/operations/components/setup-bindings/supported-bindings/kafka/"
---

## 配置

要设置 Kafka 绑定，请创建一个类型为 `bindings.kafka`的组件。 请参阅[本指南]({{< ref "howto-bindings.md#1-create-a-binding" >}})，了解如何创建和应用绑定配置。 有关使用 `secretKeyRef`的详细信息，请参阅有[关如何在组件中引用Secret指南]({{< ref component-secrets.md >}})。

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: kafka-binding
  namespace: default
spec:
  type: bindings.kafka
  version: v1
  metadata:
  - name: topics # Optional. Used for input bindings.
    value: "topic1,topic2"
  - name: brokers # Required.
    value: "localhost:9092,localhost:9093"
  - name: consumerGroup # Optional. Used for input bindings.
    value: "group1"
  - name: publishTopic # Optional. Used for output bindings.
    value: "topic3"
  - name: authRequired # Required.
    value: "true"
  - name: saslUsername # Required if authRequired is `true`.
    value: "user"
  - name: saslPassword # Required if authRequired is `true`.
    secretKeyRef:
      name: kafka-secrets
      key: saslPasswordSecret
  - name: initialOffset # Optional. Used for input bindings.
    value: "newest"
  - name: maxMessageBytes # Optional.
    value: 1024
  - name: version # Optional.
    value: 1.0.0
```

## 元数据字段规范

| 字段                  | 必填 | 绑定支持  | 详情                                                                                                                    | 示例                                                         |
| ------------------- |:--:| ----- | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| topics              | 否  | 输入    | 以逗号分隔的主题字符串。                                                                                                          | `"mytopic1,topic2"`                                        |
| brokers             | 是  | 输入/输出 | 以逗号分隔的 Kafka broker。                                                                                                  | `"localhost:9092,dapr-kafka.myapp.svc.cluster.local:9093"` |
| clientID            | 否  | 输入/输出 | 用户提供的字符串，随每个请求一起发送到 Kafka 代理，用于日志记录、调试和审计目的。                                                                          | `"my-dapr-app"`                                            |
| consumerGroup       | 否  | 输入    | 监听 kafka 消费者组。 发布到主题的每条记录都会传递给订阅该主题的每个消费者组中的一个消费者。                                                                    | `"group1"`                                                 |
| consumeRetryEnabled | 否  | 输入/输出 | 通过设置为 `"true"`启用消费重试。 在 Kafka 绑定组件中默认为 `false`。                                                                       | `"true"`, `"false"`                                        |
| publishTopic        | 是  | 输出    | 要发布的主题。                                                                                                               | `"mytopic"`                                                |
| authRequired        | 否  | *已废弃* | 启用 [SASL](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) 对 Kafka broker 的身份验证。               | `"true"`, `"false"`                                        |
| authType            | 是  | 输入/输出 | 配置或禁用身份验证。 支持值包括： `none`, `password`, `mtls`, 或者 `oidc`                                                               | `"password"`, `"none"`                                     |
| saslUsername        | 否  | 输入/输出 | 用于身份验证的 SASL 用户名。 仅当 `authRequired` 设置为 `"true"`时才需要。                                                                 | `"adminuser"`                                              |
| saslPassword        | 否  | 输入/输出 | 用于身份验证的 SASL 密码。 可以用`secretKeyRef`来[引用 Secret]({{< ref component-secrets.md >}})。 仅当 `authRequired` 设置为 `"true"`时才需要。 | `""`, `"KeFg23!"`                                          |
| initialOffset       | 否  | 输入    | 如果以前未提交任何偏移量，则要使用的初始偏移量。 应为"newest"或"oldest"。 默认为"newest"。                                                            | `"oldest"`                                                 |
| maxMessageBytes     | 否  | 输入/输出 | 单条Kafka消息允许的最大消息的字节大小。 默认值为 1024。                                                                                     | `2048`                                                     |
| oidcTokenEndpoint   | 否  | 输入/输出 | OAuth2 身份提供者访问令牌端点的完整 URL。 将`authType`的值设置为`oidc`时需要设置。                                                               | "https://identity.example.com/v1/token"                    |
| oidcClientID        | 否  | 输入/输出 | 已在标识提供者中预配的 OAuth2 客户端 ID。 将`authType`的值设置为`oidc`时需要设置。                                                               | `dapr-kafka`                                               |
| oidcClientSecret    | 否  | 输入/输出 | 已在身份提供者中配置的 OAuth2 客户端密码：当 `authType` 设置为 `oidc`时需要                                                                   | `"KeFg23!"`                                                |
| oidcScopes          | 否  | 输入/输出 | 使用访问令牌请求的 OAuth2/OIDC 范围的逗号分隔列表。 当 `authType` 设置为 `oidc`时推荐使用。 默认值为 `"openid"`                                        | `"openid,kafka-prod"`                                      |

## 绑定支持

此组件支持 **输入和输出** 绑定接口。

该组件支持如下操作的 **输出绑定** ：

- `create`

## 鉴权

Kafka 支持多种身份验证模式，Dapr 支持几种：SASL 密码、mTLS、OIDC/OAuth2。 [详细更多关于 Kafka 对 Kafka 绑定和 Kafka 发布/订阅组件]({{< ref "setup-apache-kafka.md#authentication" >}})的身份验证方法。

## 指定分区键

调用 Kafka 绑定时，可以使用请求正文中的 `metadata` 部分提供可选的分区键。

字段名称为 `partitionKey`。

示例:

```shell
curl -X POST http://localhost:3500/v1.0/bindings/myKafka \
  -H "Content-Type: application/json" \
  -d '{
        "data": {
          "message": "Hi"
        },
        "metadata": {
          "partitionKey": "key1"
        },
        "operation": "create"
      }'
```

### 响应

请求成功，将返回HTTP 204状态码(无内容) 和空报文

## 相关链接

- [Dapr组件的基本格式]({{< ref component-schema >}})
- [绑定构建块]({{< ref bindings >}})
- [如何通过输入绑定触发应用]({{< ref howto-triggers.md >}})
- [如何处理: 使用绑定对接外部资源]({{< ref howto-bindings.md >}})
- [Bindings API 引用]({{< ref bindings_api.md >}})
