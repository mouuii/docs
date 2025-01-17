---
type: docs
title: "RocketMQ"
linkTitle: "RocketMQ"
description: "Detailed documentation on the RocketMQ pubsub component"
aliases:
  - "/zh-hans/operations/components/setup-pubsub/supported-pubsub/setup-rocketmq/"
---

## Component format
To setup RocketMQ pubsub, create a component of type `pubsub.rocketmq`. See [this guide]({{< ref "howto-publish-subscribe.md#step-1-setup-the-pubsub-component" >}}) on how to create and apply a pubsub configuration.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: rocketmq-pubsub
spec:
  type: pubsub.rocketmq
  version: v1
  metadata:
    - name: instanceName
      value: dapr-rocketmq-test
    - name: consumerGroup
      value: dapr-rocketmq-test-g-c
    - name: producerGroup 
      value: dapr-rocketmq-test-g-p
    - name: nameSpace
      value: dapr-test
    - name: nameServer
      value: "127.0.0.1:9876,127.0.0.2:9876"
    - name: retries
      value: 3
    - name: consumerModel
      value: "clustering"
    - name: consumeOrderly
      value: false
```

{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件，参考 [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

## 元数据字段规范
| Field                                 | 必填 | 详情                                                                                                                                                                  | default                                                     | 示例                                                                            |
| ------------------------------------- |:--:| ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------------------- |
| instanceName                          | 否  | Instance name                                                                                                                                                       | `time.Now().String()`                                       | `dapr-rocketmq-test`                                                          |
| consumerGroup                         | 否  | Consumer group name. Recommend. If `producerGroup` is `null`，`groupName` is used.                                                                                   |                                                             | `dapr-rocketmq-test-g-c`                                                      |
| producerGroup (consumerID)            | 否  | Producer group name. Recommended. If `producerGroup` is `null`，`consumerID` is used. If `consumerID` also is null, `groupName` is used.                             |                                                             | `dapr-rocketmq-test-g-p`                                                      |
| groupName                             | 否  | Consumer/Producer group name. **Depreciated**.                                                                                                                      |                                                             | `dapr-rocketmq-test-g`                                                        |
| nameSpace                             | 否  | RocketMQ namespace                                                                                                                                                  |                                                             | `dapr-rocketmq`                                                               |
| nameServerDomain                      | 否  | RocketMQ name server domain                                                                                                                                         |                                                             | `https://my-app.net:8080/nsaddr`                                              |
| nameServer                            | 否  | RocketMQ name server, separated by "," or ";"                                                                                                                       |                                                             | `127.0.0.1:9876;127.0.0.2:9877,127.0.0.3:9877`                                |
| accessKey                             | 否  | Access Key (Username)                                                                                                                                               |                                                             | `"admin"`                                                                     |
| secretKey                             | 否  | Secret Key (Password)                                                                                                                                               |                                                             | `"password"`                                                                  |
| securityToken                         | 否  | Security Token                                                                                                                                                      |                                                             |                                                                               |
| retries                               | 否  | Number of retries to send a message to broker                                                                                                                       | `3`                                                         | `3`                                                                           |
| producerQueueSelector (queueSelector) | 否  | Producer Queue selector. There are five implementations of queue selector: `hash`, `random`, `manual`, `roundRobin`, `dapr`.                                        | `dapr`                                                      | `hash`                                                                        |
| consumerModel                         | 否  | Message model that defines how messages are delivered to each consumer client. RocketMQ supports two message models: `clustering` and `broadcasting`.               | `clustering`                                                | `broadcasting` , `clustering`                                                 |
| fromWhere (consumeFromWhere)          | 否  | Consuming point on consumer booting. There are three consuming points: `CONSUME_FROM_LAST_OFFSET`, `CONSUME_FROM_FIRST_OFFSET`, `CONSUME_FROM_TIMESTAMP`            | `CONSUME_FROM_LAST_OFFSET`                                  | `CONSUME_FROM_LAST_OFFSET`                                                    |
| consumeOrderly                        | 否  | Determines if it's an ordered message using FIFO order.                                                                                                             | `false`                                                     | `false`                                                                       |
| consumeMessageBatchMaxSize            | 否  | Batch consumption size out of range `[1, 1024]`                                                                                                                     | `512`                                                       | `10`                                                                          |
| maxReconsumeTimes                     | 否  | Max re-consume times. `-1` means 16 times. If messages are re-consumed more than {@link maxReconsumeTimes} before success, they'll be directed to a deletion queue. | Orderly message is `MaxInt32`; Concurrently message is `16` | `16`                                                                          |
| autoCommit                            | 否  | Enable auto commit                                                                                                                                                  | `true`                                                      | `false`                                                                       |
| pullInterval                          | 否  | Message pull interval                                                                                                                                               | `100`                                                       | `100`                                                                         |
| pullBatchSize                         | 否  | The number of messages pulled from the broker at a time. If `pullBatchSize` is `null`, use `ConsumerBatchSize`. `pullBatchSize` out of range `[1, 1024]`            | `32`                                                        | `10`                                                                          |
| content-type                          | 否  | Message content type.                                                                                                                                               | `"text/plain"`                                              | `"application/cloudevents+json; charset=utf-8"`, `"application/octet-stream"` |
| logLevel                              | 否  | Log level                                                                                                                                                           | `warn`                                                      | `info`                                                                        |
| sendTimeOut                           | 否  | Send message timeout to connect RocketMQ's broker, measured in nanoseconds. **Deprecated**.                                                                         | 3 seconds                                                   | `10000000000`                                                                 |
| sendTimeOutSec                        | 否  | Timeout duration for publishing a message in seconds. If `sendTimeOutSec` is `null`, `sendTimeOut` is used.                                                         | 3 seconds                                                   | `3`                                                                           |
| mspProperties                         | 否  | The RocketMQ message properties in this collection are passed to the APP in Data Separate multiple properties with ","                                              |                                                             | `key,mkey`                                                                    |

For backwards-compatibility reasons, the following values in the metadata are supported, although their use is discouraged.

| Field (supported but deprecated) | 必填 | 详情                                                       | 示例                       |
| -------------------------------- |:--:| -------------------------------------------------------- | ------------------------ |
| groupName                        | 否  | Producer group name for RocketMQ publishers              | `"my_unique_group_name"` |
| sendTimeOut                      | 否  | Timeout duration for publishing a message in nanoseconds | `0`                      |
| consumerBatchSize                | 否  | The number of messages pulled from the broker at a time  | `32`                     |

## Setup RocketMQ
See https://rocketmq.apache.org/docs/quick-start/ to setup a local RocketMQ instance.

## Per-call metadata fields

### Partition Key

When invoking the RocketMQ pub/sub, it's possible to provide an optional partition key by using the `metadata` query param in the request url.

You need to specify `rocketmq-tag` , `"rocketmq-key"` , `rocketmq-shardingkey` , `rocketmq-queue` in `metadata`

示例︰

```shell
curl -X POST http://localhost:3500/v1.0/publish/myRocketMQ/myTopic?metadata.rocketmq-tag=?&metadata.rocketmq-key=?&metadata.rocketmq-shardingkey=key&metadata.rocketmq-queue=1 \
  -H "Content-Type: application/json" \
  -d '{
        "data": {
          "message": "Hi"
        }
      }'
```

## QueueSelector

The RocketMQ component contains a total of five queue selectors. The RocketMQ client provides the following queue selectors:
- `HashQueueSelector`
- `RandomQueueSelector`
- `RoundRobinQueueSelector`
- `ManualQueueSelector`

To learn more about these RocketMQ client queue selectors, read the [RocketMQ documentation](https://rocketmq.apache.org/docs).

The Dapr RocketMQ component implements the following queue selector:
- `DaprQueueSelector`

 This article focuses on the design of `DaprQueueSelector`.

### DaprQueueSelector

`DaprQueueSelector` integrates three queue selectors:
- `HashQueueSelector`
- `RoundRobinQueueSelector`
- `ManualQueueSelector`

`DaprQueueSelector` gets the queue id from the request parameter. You can set the queue id by running the following:

```
http://localhost:3500/v1.0/publish/myRocketMQ/myTopic?metadata.rocketmq-queue=1
```

The `ManualQueueSelector` is implemented using the method above.

Next, the `DaprQueueSelector` tries to:
- Get a `ShardingKey`
- Hash the `ShardingKey` to determine the queue id.

You can set the `ShardingKey` by doing the following:

```
http://localhost:3500/v1.0/publish/myRocketMQ/myTopic?metadata.rocketmq-shardingkey=key
```

If the `ShardingKey` does not exist, the `RoundRobin` algorithm is used to determine the queue id.

## 相关链接

- [Basic schema for a Dapr component]({{< ref component-schema >}})
- [发布/订阅构建块]({{< ref pubsub >}})
- 阅读 [本指南]({{< ref "howto-publish-subscribe.md#step-2-publish-a-topic" >}})，了解配置 发布/订阅组件的说明
