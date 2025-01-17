---
type: docs
title: "Zookeeper"
linkTitle: "Zookeeper"
description: Zookeeper 状态存储组件的详细信息
aliases:
  - "/zh-hans/operations/components/setup-state-store/supported-state-stores/setup-zookeeper/"
---

## Component format

To setup Zookeeper state store create a component of type `state.zookeeper`. See [this guide]({{< ref "howto-get-save-state.md#step-1-setup-a-state-store" >}}) on how to create and apply a state store configuration.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <NAME>
spec:
  type: state.zookeeper
  version: v1
  metadata:
  - name: servers
    value: <REPLACE-WITH-COMMA-DELIMITED-SERVERS> # Required. Example: "zookeeper.default.svc.cluster.local:2181"
  - name: sessionTimeout
    value: <REPLACE-WITH-SESSION-TIMEOUT> # Required. Example: "5s"
  - name: maxBufferSize
    value: <REPLACE-WITH-MAX-BUFFER-SIZE> # Optional. default: "1048576"
  - name: maxConnBufferSize
    value: <REPLACE-WITH-MAX-CONN-BUFFER-SIZE> # Optional. default: "1048576"
  - name: keyPrefixPath
    value: <REPLACE-WITH-KEY-PREFIX-PATH> # Optional.
```

{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件，参考 [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

## 元数据字段规范

| Field             | 必填 | 详情                                | 示例                                           |
| ----------------- |:--:| --------------------------------- | -------------------------------------------- |
| servers           | 是  | Comma delimited list of servers   | `"zookeeper.default.svc.cluster.local:2181"` |
| sessionTimeout    | 是  | 会话超时值                             | `"5s"`                                       |
| maxBufferSize     | 否  | 缓冲区的最大大小。 Defaults to `"1048576"` | `"1048576"`                                  |
| maxConnBufferSize | 否  | 连接缓冲区的最大大小。 默认为 `"1048576"`       | `"1048576"`                                  |
| keyPrefixPath     | 否  | Zookeeper 中的键前缀路径。 No default     | `"dapr"`                                     |

## 设置 Zookeeper

{{< tabs "Self-Hosted" "Kubernetes" >}}

{{% codetab %}}
您可以使用 Docker 在本地运行 Zookeeper：

```
docker run --name some-zookeeper --restart always -d zookeeper
```

You can then interact with the server using `localhost:2181`.
{{% /codetab %}}

{{% codetab %}}
在 Kubernetes 上安装 Zookeeper 最简单的方法方法是使用 [Helm chart](https://github.com/helm/charts/tree/master/incubator/zookeeper)：

```
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
helm install zookeeper incubator/zookeeper
```

这将会把 Zookeeper 安装到 `default` 命名空间。 要与 Zookeeper 交互，使用 `kubectl get svc zookeeper` 找到 service。

例如，如果你使用上面的示例安装，Zookeeper 的主机地址将是：

`zookeeper.default.svc.cluster.local:2181`
{{% /codetab %}}

{{< /tabs >}}


## 相关链接
- [Basic schema for a Dapr component]({{< ref component-schema >}})
- 阅读 [本指南]({{< ref "howto-get-save-state.md#step-2-save-and-retrieve-a-single-state" >}}) 以获取配置状态存储组件的说明
- [状态管理构建块]({{< ref state-management >}})
