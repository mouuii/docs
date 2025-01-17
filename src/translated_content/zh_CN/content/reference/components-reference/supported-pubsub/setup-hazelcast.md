---
type: docs
title: "Hazelcast"
linkTitle: "Hazelcast"
description: "关于Hazelcast pubsub组件的详细文档。"
aliases:
  - "/zh-hans/operations/components/setup-pubsub/supported-pubsub/setup-hazelcast/"
---

## 配置
要安装 hazelcast pubsub ，请创建一个类型为 `pubsub.hazelcast` 的组件。 请参阅[本指南]({{< ref "howto-publish-subscribe.md#step-1-setup-the-pubsub-component" >}})，了解如何创建和应用 pubsub 配置。

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: hazelcast-pubsub
  namespace: default
spec:
  type: pubsub.hazelcast
  version: v1
  metadata:
  - name: hazelcastServers
    value: "hazelcast:3000,hazelcast2:3000"
```

{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件， [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

## 元数据字段规范

| 字段                | 必填 | 详情                                                                                                        | 示例                                 |
| ----------------- |:--:| --------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| connectionString  | 是  | 逗号分隔的服务器地址 逗号分隔的服务器地址 示例：“hazelcast:3000,hazelcast2:3000”                                                 | `"hazelcast:3000,hazelcast2:3000"` |
| backOffMaxRetries | 否  | 返回错误前重试处理消息的最大次数。 默认为 `"0"` 这意味着组件不会重试处理消息。 `"-1"` 将无限期重试，直到处理完消息或关闭应用程序。 并且正数被视为最大重试次数。 组件在每次重试之前将等待5秒钟。 | `"3"`                              |


## 创建Hazelcast 实例

{{< tabs "Self-Hosted" "Kubernetes">}}

{{% codetab %}}
你可以使用Docker在本地运行Hazelcast：

```
docker run -e JAVA_OPTS="-Dhazelcast.local.publicAddress=127.0.0.1:5701" -p 5701:5701 hazelcast/hazelcast
```

然后你可以通过`127.0.0.1:5701`与服务器交互。
{{% /codetab %}}

{{% codetab %}}
在Kubernetes上安装Hazelcast的最简单方法是使用[Helm chart](https://github.com/helm/charts/tree/master/stable/hazelcast)。
{{% /codetab %}}

{{< /tabs >}}

## 相关链接
- [Dapr组件的基本格式]({{< ref component-schema >}})
- 阅读 [本指南]({{< ref "howto-publish-subscribe.md#step-2-publish-a-topic" >}})，了解配置 发布/订阅组件的说明
- [发布/订阅构建块]({{< ref pubsub >}})