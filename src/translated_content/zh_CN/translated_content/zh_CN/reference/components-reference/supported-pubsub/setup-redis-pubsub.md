---
type: docs
title: "Redis Streams"
linkTitle: "Redis Streams"
description: "关于Redis Streams pubsub组件的详细文档"
aliases:
  - "/zh-hans/operations/components/setup-pubsub/supported-pubsub/setup-redis-pubsub/"
---

## Component format

To setup Redis Streams pubsub create a component of type `pubsub.redis`. See [this guide]({{< ref "howto-publish-subscribe.md#step-1-setup-the-pubsub-component" >}}) on how to create and apply a pubsub configuration.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: redis-pubsub
spec:
  type: pubsub.redis
  version: v1
  metadata:
  - name: redisHost
    value: localhost:6379
  - name: redisPassword
    value: "KeFg23!"
  - name: consumerID
    value: "myGroup"
  - name: enableTLS
    value: "false"
```

{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件，参考 [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

## 元数据字段规范

| Field                 | 必填 | 详情                                                                                                                                       | 示例                                                              |
| --------------------- |:--:| ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| redisHost             | 是  | Connection-string for the redis host. If `"redisType"` is `"cluster"` it can be multiple hosts separated by commas or just a single host | `localhost:6379`, `redis-master.default.svc.cluster.local:6379` |
| redisPassword         | 是  | Redis的密码 无默认值 可以用`secretKeyRef`来引用密钥。                                                                                                    | `""`, `"KeFg23!"`                                               |
| redisUsername         | 否  | Redis 主机的用户名。 默认为空. 确保您的 redis 服务器版本为 6 或更高版本，并且已正确创建 acl 规则。                                                                            | `""`, `"default"`                                               |
| consumerID            | 否  | The consumer group ID                                                                                                                    | `"myGroup"`                                                     |
| enableTLS             | 否  | 如果Redis实例支持使用公共证书的TLS，可以配置为启用或禁用。 默认值为 `"false"`                                                                                         | `"true"`, `"false"`                                             |
| redeliverInterval     | 否  | 检查待处理消息到重发的间隔。 默认为 `"60s"`. `"0"` 禁用重发。                                                                                                  | `"30s"`                                                         |
| processingTimeout     | 否  | 在尝试重新发送消息之前必须等待的时间。 默认为 `"15s"`。 `"0"` 禁用重发。                                                                                             | `"30s"`                                                         |
| queueDepth            | 否  | 用于处理的消息队列的大小。 Defaults to `"100"`.                                                                                                       | `"1000"`                                                        |
| concurrency           | 否  | 正在处理消息的并发工作线程数。 Defaults to `"10"`.                                                                                                      | `"15"`                                                          |
| redisType             | 否  | Redis 的类型。 有两个有效的值，一个是 `"node"` 用于单节点模式，另一个是 `"cluster"` 用于 redis 集群模式。 默认为 `"node"`。                                                    | `"cluster"`                                                     |
| redisDB               | 否  | 连接到 redis 后选择的数据库。 如果 `"redisType"` 是 `"cluster "` 此选项被忽略。 Defaults to `"0"`.                                                            | `"0"`                                                           |
| redisMaxRetries       | 否  | 放弃前重试命令的最大次数。 默认值为不重试失败的命令。                                                                                                              | `"5"`                                                           |
| redisMinRetryInterval | 否  | 每次重试之间 redis 命令的最小回退时间。 默认值为 `"8ms"`;  `"-1"` 禁用回退。                                                                                      | `"8ms"`                                                         |
| redisMaxRetryInterval | 否  | 每次重试之间 redis 命令的最大回退时间。 默认值为 `"512ms"`;`"-1"` 禁用回退。                                                                                      | `"5s"`                                                          |
| dialTimeout           | 否  | 建立新连接的拨号超时。 默认为 `"5s"`。                                                                                                                  | `"5s"`                                                          |
| readTimeout           | 否  | 套接字读取超时。 如果达到，redis命令将以超时的方式失败，而不是阻塞。 默认为 `"3s"`, `"-1"` 表示没有超时。                                                                         | `"3s"`                                                          |
| writeTimeout          | 否  | 套接字写入超时。 如果达到，redis命令将以超时的方式失败，而不是阻塞。 默认值为 readTimeout。                                                                                  | `"3s"`                                                          |
| poolSize              | 否  | 最大套接字连接数。 默认是每个CPU有10个连接，由 runtime.NumCPU 所述。                                                                                            | `"20"`                                                          |
| poolTimeout           | 否  | 如果所有连接都处于繁忙状态，客户端等待连接时间，超时后返回错误。 默认值为 readTimeout + 1 秒。                                                                                 | `"5s"`                                                          |
| maxConnAge            | 否  | 客户端退出（关闭）连接时的连接期限。 默认值是不关闭过期的连接。                                                                                                         | `"30m"`                                                         |
| minIdleConns          | 否  | 保持开放的最小空闲连接数，以避免创建新连接带来的性能下降。 Defaults to `"0"`.                                                                                         | `"2"`                                                           |
| idleCheckFrequency    | 否  | 空闲连接后的空闲检查频率。 默认值为 `"1m"`。 `"-1"` 禁用空闲连接回收。                                                                                              | `"-1"`                                                          |
| idleTimeout           | 否  | 客户端关闭空闲连接的时间量。 应小于服务器的超时。 默认值为 `"5m"`。 `"-1"` 禁用空闲超时检查。                                                                                  | `"10m"`                                                         |
| failover              | 否  | 已启用故障转移配置的属性。 需要设置 sentinalMasterName。 默认值为 `"false"`                                                                                    | `"true"`, `"false"`                                             |
| sentinelMasterName    | 否  | 哨兵主名称。 See [Redis Sentinel Documentation](https://redis.io/docs/manual/sentinel/)                                                        | `""`,  `"127.0.0.1:6379"`                                       |
| maxLenApprox          | 否  | 流中的最大项目数。当达到指定长度时，将自动逐出旧条目，以便流保持恒定大小。 默认为无限制。                                                                                            | `"10000"`                                                       |

## 创建Redis实例

Dapr can use any Redis instance - containerized, running on your local dev machine, or a managed cloud service, provided the version of Redis is 5.x or 6.x.

{{< tabs "Self-Hosted" "Kubernetes" "AWS" "GCP" "Azure">}}

{{% codetab %}}
Dapr CLI将自动为你创建和设置一个Redis Streams实例。 当你执行`dapr init`时，Redis实例将通过Docker安装，组件文件将在默认目录下创建。 (默认目录位于`$HOME/.dapr/components` (Mac/Linux) ，`%USERPROFILE%\.dapr\components` (Windows)).
{{% /codetab %}}

{{% codetab %}}
你可以使用[Helm](https://helm.sh/)在我们的Kubernetes集群中快速创建一个Redis实例， 这种方法需要[安装Helm](https://github.com/helm/helm#install)。 这种方法需要[安装Helm](https://github.com/helm/helm#install)。

1. Install Redis into your cluster.
    ```bash
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm install redis bitnami/redis --set image.tag=6.2
    ```

2. 执行`kubectl get pods`来查看现在正在集群中运行的Redis容器。
3. 在您的redis.yaml文件中添加`redis-master:6379`作为`redisHost`。 例如:

    ```yaml
        metadata:
        - name: redisHost
          value: redis-master:6379
    ```

4. 接下来，我们会获取到我们的Redis密码，根据我们使用的操作系统不同，密码也会略有不同：
    - **Windows**：执行`kubectl get secret --namespace default redis -o jsonpath="{.data.redis-password}" > encoded.b64`，这将创建一个有你的加密后密码的文件。 接下来，执行`certutil -decode encoded.b64 password.txt`，它将把你的redis密码放在一个名为`password.txt`的文本文件中。 复制密码，删除这两个文件。

    - **Linux/MacOS**: Run `kubectl get secret --namespace default redis -o jsonpath="{.data.redis-password}" | base64 --decode` and copy the outputted password.

    Add this password as the `redisPassword` value in your redis.yaml file. For example:

    ```yaml
        - name: redisPassword
          value: "lhDOkwTlp0"
    ```
{{% /codetab %}}

{{% codetab %}}
[AWS Redis](https://aws.amazon.com/redis/)
{{% /codetab %}}

{{% codetab %}}
[GCP Cloud MemoryStore](https://cloud.google.com/memorystore/)
{{% /codetab %}}

{{% codetab %}}
[Azure Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/quickstart-create-redis)
{{% /codetab %}}

{{< /tabs >}}


{{% alert title="Note" color="primary" %}}
作为`dapr init`命令的一部分，Dapr CLI会在自托管模式下自动部署本地redis实例。
{{% /alert %}}

## 相关链接
- [Basic schema for a Dapr component]({{< ref component-schema >}})
- 阅读 [本指南]({{< ref "howto-publish-subscribe.md#step-2-publish-a-topic" >}})，了解配置 发布/订阅组件的说明
- [发布/订阅构建块]({{< ref pubsub >}})