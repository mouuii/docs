---
type: docs
title: "RabbitMQ"
linkTitle: "RabbitMQ"
description: "RabbitMQ pubsub组件的详细文档"
aliases:
  - "/zh-hans/operations/components/setup-pubsub/supported-pubsub/setup-rabbitmq/"
---

## 配置

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: rabbitmq-pubsub
  namespace: default
spec:
  type: pubsub.rabbitmq
  version: v1
  metadata:
  - name: host
    value: "amqp://localhost:5672"
  - name: consumerID
    value: myapp
  - name: durable
    value: false
  - name: deletedWhenUnused
    value: false
  - name: autoAck
    value: false
  - name: deliveryMode
    value: 0
  - name: requeueInFailure
    value: false
  - name: prefetchCount
    value: 0
  - name: reconnectWait
    value: 0
  - name: concurrencyMode
    value: parallel
  - name: backOffPolicy
    value: exponential
  - name: backOffInitialInterval
    value: 100
  - name: backOffMaxRetries
    value: 16
  - name: enableDeadLetter # Optional enable dead Letter or not
    value: true
  - name: maxLen # Optional max message count in a queue
    value: 3000
  - name: maxLenBytes # Optional maximum length in bytes of a queue.
    value: 10485760
  - name: exchangeKind
    value: fanout
```
{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件， [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

## 元数据字段规范

| 字段                         | 必填 | 详情                                                                                                                                                  | 示例                                |
| -------------------------- |:--:| --------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| host                       | 是  | Rabbitmq 的连接地址                                                                                                                                      | `amqp://user:pass@localhost:5672` |
| consumerID                 | 否  | 消费者ID，也叫做消费者标签，将一个或多个消费者组成一个组。 具有相同消费者 ID 的消费者作为一个虚拟消费者工作，即消息仅由组中的消费者处理一次。 如果未设置使用者 ID，Dapr 运行时会将其设置为 Dapr 应用程序 ID 。                                |                                   |
| durable                    | 否  | 是否使用[durable](https://www.rabbitmq.com/queues.html#durability)队列， 默认值为 `"false"` 默认值为 `"false"`                                                     | `"true"`, `"false"`               |
| deletedWhenUnused          | 否  | 是否应将队列配置为 [自动删除](https://www.rabbitmq.com/queues.html) 默认值为 `"true"`                                                                                | `"true"`, `"false"`               |
| autoAck                    | 否  | 队列的消费者是否应该[auto-ack](https://www.rabbitmq.com/confirms.html)消息 默认值为 `"false"` 默认值为 `"false"`                                                        | `"true"`, `"false"`               |
| deliveryMode               | 否  | 发布消息时的持久化模式， 默认值为 `"0"`. 值为`"2"`时RabbitMQ会进行持久化，其他值反之                                                                                               | `"0"`, `"2"`                      |
| requeueInFailure           | 否  | 在发送[否定应答](https://www.rabbitmq.com/nack.html)失败的情况下，是否进行重排。 默认值为 `"false"`                                                                          | `"true"`, `"false"`               |
| prefetchCount              | 否  | [prefecth](https://www.rabbitmq.com/consumer-prefetch.html)的消息数量。 生产环境中需要考虑设置一个非零值。 该值默认为`"0"`，这意味着所有可用消息都将被预先提取                                    | `"2"`                             |
| reconnectWait              | 否  | 如果发生连接失败，在重新连接之前需要等待多长时间（秒）                                                                                                                         | `"0"`                             |
| concurrencyMode            | 否  | 默认值是`parallel`，表示允许并行处理多个消息（如果配置了`app-max-concurrency`，最大并行数会受到该值限制）, 设置为`single`可禁用并行处理， 大多数情况下没必要去这么做 设置为`single`可禁用并行处理， 大多数情况下没必要去这么做           | `parallel`, `single`              |
| backOffPolicy              | 否  | 重试策略，`"constant"`是一个总是返回相同的退避延迟的退避策略。 `"exponential"` 是一种退避策略，它使用呈指数级增长的随机化函数增加每次重试尝试的退避周期。 默认为 `"constant"`。                                       | `constant`、`exponential`          |
| backOffDuration            | 否  | 固定间隔仅在策略恒定时生效。 有两种有效的格式，一种是带有单位后缀格式的分数，另一种是将以毫秒为单位处理的纯数字格式。 有效的时间单位为"ns"、"us"（或"μs"）、"ms"、"s"、"m"、"h"。 默认为 `"5s"`。                                  | `"5s"`、`"5000"`                   |
| backOffInitialInterval     | 否  | 重试时的回退初始间隔。 仅当策略是指数型时才生效。 有两种有效的格式，一种是带有单位后缀格式的分数，另一种是将以毫秒为单位处理的纯数字格式。 有效的时间单位为"ns"、"us"（或"μs"）、"ms"、"s"、"m"、"h"。 默认值为 `"500"`                      | `"50"`                            |
| backOffMaxInterval         | 否  | 重试时的回退初始间隔。 仅当策略是指数型时才生效。 有两种有效的格式，一种是带有单位后缀格式的分数，另一种是将以毫秒为单位处理的纯数字格式。 有效的时间单位为"ns"、"us"（或"μs"）、"ms"、"s"、"m"、"h"。 默认为 `"60s"`                       | `"60000"`                         |
| backOffMaxRetries          | 否  | 返回错误前重试处理消息的最大次数。 默认为 `"0"` 这意味着组件不会重试处理消息。 `"-1"` 将无限期重试，直到处理完消息或关闭应用程序。 任何正数都被视为最大重试计数。                                                           | `"3"`                             |
| backOffRandomizationFactor | 否  | 随机系数，介于 1 和 0 之间，包括 0 但不是 1。 随机间隔 = 重试间隔 * （1 ±backOffRandomizationFactor）。 默认值为 `"0.5"`.                                                           | `"0.5"`                           |
| backOffMultiplier          | 否  | 策略的退避倍数。 通过将间隔乘以倍数来递增间隔。 默认值为 `"1.5"`                                                                                                               | `"1.5"`                           |
| backOffMaxElapsedTime      | 否  | 在 MaxElapsedTime 之后，ExponentialBackOff 返回 Stop。 有两种有效的格式，一种是带有单位后缀格式的分数，另一种是将以毫秒为单位处理的纯数字格式。 有效的时间单位为"ns"、"us"（或"μs"）、"ms"、"s"、"m"、"h"。 默认为 `"15m"` | `"15m"`                           |
| enableDeadLetter           | 否  | 启用转发功能 不能处理的信息会被转发到一个死信主题。 默认值为 `"false"`                                                                                                           | `"true"`, `"false"`               |
| maxLen                     | 否  | 队列及其死信队列的最大消息数（如果启用了死信）。 如果同时设置了 `maxLen` 和 `maxLenBytes` ，则两者都适用; 将强制执行首先达到的限制。  默认为无限制。                                                           | `"1000"`                          |
| maxLenBytes                | 否  | 队列及其死信队列的最大长度（以字节为单位）（如果启用了死信）。 如果同时设置了 `maxLen` 和 `maxLenBytes` ，则两者都适用; 将强制执行首先达到的限制。  默认为无限制。                                                    | `"1048576"`                       |
| exchangeKind               | 否  | Rabbitmq交换机的类型  默认为 `"fanout"`。                                                                                                                     | `"fanout"`,`"topic"`              |


### 退避策略介绍
回退重试策略可以指示 dapr sidecar 如何重新发送消息。 默认情况下，重试策略处于关闭状态，这意味着 sidecar 将向服务发送一次消息。 当服务返回结果时，无论消息是否正确，该消息都将被标记为已消费。 以上是基于 `autoAck` 的条件， `requeueInFailure` 设置为false（如果 `requeueInFailure` 设置为true，则消息将获得第二次机会）。

但在某些情况下，您可能希望 dapr 使用（指数或常量）回退策略重试推送消息，直到消息正常处理或重试次数用尽。 当您的服务异常发生故障但 sidecar 没有一起停止时，这可能很有用。 添加回退策略将在服务停机期间重试消息推送，而不是将这些消息标记为已消费。


## 创建RabbitMQ服务

{{< tabs "Self-Hosted" "Kubernetes" >}}

{{% codetab %}}
你可以使用Docker在本地运行RabbitMQ ：

```bash
docker run -d --hostname my-rabbit --name some-rabbit rabbitmq:3
```

然后你可以通过`localhost:5672`与服务器交互。
{{% /codetab %}}

{{% codetab %}}
在 Kubernetes 上安装 RabbitMQ 最简单的方法是使用 [Helm chart](https://github.com/helm/charts/tree/master/stable/rabbitmq)。

```bash
helm install rabbitmq stable/rabbitmq
```

根据Helm图表的输出，得到用户名和密码。

这会把 RabbitMQ 安装到 `default` 命名空间中， 这会把 RabbitMQ 安装到 `default` 命名空间中， 要与RabbitMQ进行交互，请使用以下方法找到服务：`kubectl get svc rabbitmq`。

如果使用上面的示例进行安装，RabbitMQ服务器的客户端地址是：

`rabbitmq.default.svc.cluster.local:5672`
{{% /codetab %}}

{{< /tabs >}}

## 使用topic交换来路由消息
将 `exchangeKind` 设置为 `"topic"` 使用topic交换信息，通常用于消息的多播路由。 在订阅时，将根据元数据中定义的`routing key`，将具有 `routing key`的消息路由到一个或多个队列。 路由键由 ` routingKey ` 元数据定义。 例如，如果应用配置了路由键 `keyA`：
```
apiVersion: dapr.io/v1alpha1
kind: Subscription
metadata:
  name: order_pub_sub
spec:
  topic: B
  route: /B
  pubsubname: pubsub
  metadata:
    routingKey: keyA
```
它将接收带有路由键 `keyA`的消息，而不会接收带有其他路由键的消息。
```
// publish messages with routing key `keyA`, and these will be received by the above example.
client.PublishEvent(context.Background(), "pubsub", "B", []byte("this is a message"), dapr.PublishEventWithMetadata(map[string]string{"routingKey": "keyA"}))
// publish messages with routing key `keyB`, and these will not be received by the above example.
client.PublishEvent(context.Background(), "pubsub", "B", []byte("this is another message"), dapr.PublishEventWithMetadata(map[string]string{"routingKey": "keyB"}))
```

有关更多信息，请参阅 [rabbitmq 交换机](https://www.rabbitmq.com/tutorials/amqp-concepts.html#exchanges)。

## 相关链接
- 相关链接部分中的[Dapr组件的基本格式]({{< ref component-schema >}})
- 阅读 [本指南]({{< ref "howto-publish-subscribe.md#step-2-publish-a-topic" >}})，了解配置 发布/订阅组件的说明
- [发布/订阅构建块]({{< ref pubsub >}})
