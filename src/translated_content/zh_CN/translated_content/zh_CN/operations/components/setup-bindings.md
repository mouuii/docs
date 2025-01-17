---
type: docs
title: "绑定组件"
linkTitle: "绑定"
description: "关于设置 Dapr 绑定组件的指南"
weight: 900
---

Dapr integrates with external resources to allow apps to both be triggered by external events and interact with the resources. Each binding component has a name and this name is used when interacting with the resource.

与其他构建块组件一样，绑定存储组件是可扩展的，可以在 [components-contrib 仓库](https://github.com/dapr/components-contrib)中找到。

在 Dapr 中描述的绑定使用了 `Component` 文件，具有以下字段：

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <NAME>
  namespace: <NAMESPACE>
spec:
  type: bindings.<NAME>
  version: v1
  metadata:
  - name: <KEY>
    value: <VALUE>
  - name: <KEY>
    value: <VALUE>
...
```

绑定类型由 `type` 字段确定，连接字符串和其他元数据等内容放在 `.metadata` 部分中。

不同的 [支持的绑定]({{< ref supported-bindings >}}) 将有不同的特定字段需要配置。 例如，当配置 [Azure Blob Storage]({{< ref blobstorage>}}) 的绑定时，文件看起来就像这样：

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <NAME>
spec:
  type: bindings.azure.blobstorage
  version: v1
  metadata:
  - name: storageAccount
    value: myStorageAccountName
  - name: storageAccessKey
    value: ***********
  - name: container
    value: container1
  - name: decodeBase64
    value: <bool>
  - name: getBlobRetryCount
    value: <integer>
```

## 应用配置

创建组件的 YAML 文件后，请按照以下说明根据您的主机环境应用该文件：


{{< tabs "Self-Hosted" "Kubernetes" >}}

{{% codetab %}}
To run locally, create a `components` dir containing the YAML file and provide the path to the `dapr run` command with the flag `--resources-path`.
{{% /codetab %}}

{{% codetab %}}
若要在 Kubernetes 中部署，假定您的组件文件名为 `mybinding.yaml`，运行：

```bash
kubectl apply -f mybinding.yaml
```
{{% /codetab %}}

{{< /tabs >}}

## Supported bindings

访问 [绑定参考文档]({{< ref supported-bindings >}}) 获取支持资源的完整列表。

## 相关链接
- [绑定构建块]({{< ref bindings >}})
- [Supported bindings]({{<ref supported-bindings >}})