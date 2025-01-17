---
type: docs
title: "密钥仓库组件"
linkTitle: "Secret stores（密钥仓库）"
description: "关于配置不同的密钥仓库组件的指南"
weight: 3000
aliases:
  - "/zh-hans/operations/components/setup-state-store/secret-stores-overview/"
---

Dapr 集成secret stores，为应用程序和其他组件提供安全存储和访问Secret，如访问键和密码。 每个密钥存储组件都有一个名称，这个名称用于访问密钥。

与其他构建块组件一样，密钥存储组件是可扩展的，可以在[components-contrib 仓库](https://github.com/dapr/components-contrib)中找到。

Dapr中的密钥存储使用`Component`文件描述，其字段如下:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: secretstore
  namespace: default
spec:
  type: secretstores.<NAME>
  version: v1
  metadata:
  - name: <KEY>
    value: <VALUE>
  - name: <KEY>
    value: <VALUE>
...
```

密钥存储的类型由`type`字段决定，连接地址和其他元数据等放在`.metadata`部分。

不同[支持的 secret stores]({{< ref supported-secret-stores >}}) 将有不同的特定字段需要配置。 例如，当配置一个使用 AWS Secrets Manager秘密存储时，文件看起来就像这样：

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: awssecretmanager
  namespace: default
spec:
  type: secretstores.aws.secretmanager
  version: v1
  metadata:
  - name: region
    value: "[aws_region]"
  - name: accessKey
    value: "[aws_access_key]"
  - name: secretKey
    value: "[aws_secret_key]"
  - name: sessionToken
    value: "[aws_session_token]"
```

{{% alert title="Important" color="warning" %}}
当在 EKS (AWS Kubernetes) 上与您的应用程序一起运行 Dapr sidecar (daprd) 时，如果您使用的node/pod 已附加到定义 AWS 资源访问权限的 IAM 策略，那么您 **不能**在正在使用的组件规范的定义中提供 AWS access-key、secret-key 和token。
{{% /alert %}}

## 应用配置

一旦您创建了组件的 YAML 文件，按照以下说明来根据您的主机环境应用它：


{{< tabs "Self-Hosted" "Kubernetes" >}}

{{% codetab %}}
要在本地运行，创建一个包含YAML文件的`components`目录，并提供`dapr run`命令的路径，标志为`--components-path`。
{{% /codetab %}}

{{% codetab %}}
若要在 Kubernetes 中部署，假定您的组件文件名为 `secret-store.yaml`，运行：

```bash
kubectl apply -f secret-store.yaml
```
{{% /codetab %}}

{{< /tabs >}}

## 支持的密钥存储

访问 [ secret stores 参考]({{< ref supported-secret-stores >}}) 获取支持的secret stores完整列表。


## 相关链接

- [支持的密钥存储组件]({{< ref supported-secret-stores >}})
- [密钥构建块]({{< ref secrets >}})
