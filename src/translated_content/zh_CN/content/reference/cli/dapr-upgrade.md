---
type: docs
title: "upgrade CLI 命令参考"
linkTitle: "upgrade"
description: "有关 upgrade CLI 命令的详细信息"
---

### 说明

在支持的托管平台上升级或降级 Dapr。

{{% alert title="Warning" color="warning" %}}
在执行升级或者降级时，目标版本级别应该逐步进行，包括小版本。

在降级之前，请确认组件是向后兼容的，并且应用程序代码确实已经移除了以前版本Dapr 中不支持的 API。
{{% /alert %}}

### 支持的平台

- [Kubernetes]({{< ref kubernetes >}})

### 用法

```bash
dapr upgrade [flags]
```

### 参数

| Name                 | 环境变量 | 默认值      | 说明                                            |
| -------------------- | ---- | -------- | --------------------------------------------- |
| `--help`, `-h`       |      |          | 显示此帮助消息                                       |
| `--kubernetes`, `-k` |      | `false`  | 更新/降级 Kubernetes 集群中的 dapr                    |
| `--runtime-version`  |      | `latest` | 要升级/降级到的 Dapr 运行时版本，例如： `1.0.0`               |
| `--set`              |      |          | 在命令行上设置值 (可以用逗号指定多个或多个值: key1=val1,key2=val2) |

### 示例

```bash
# 在Kubernetes集群中，升级 Dapr到最新版本
dapr upgrade -k

# 在Kubernetes集群中，升级或降级 Dapr runtime 到一个指定版本
dapr upgrade -k --runtime-version 1.2

# 在Kubernetes集群中，升级或降级 Dapr runtime 到一个指定版本，并且进行值设定
dapr upgrade -k --runtime-version 1.2 --set global.logAsJson=true
```
### 警告信息
此命令可以发出警告消息。

#### 根证书续订警告
如果部署到 Kubernetes 集群的 mtls 根证书在 30 天内过期，则会显示以下警告消息：

```
Dapr root certificate of your Kubernetes cluster expires in <n> days. Expiry date: <date:time> UTC. 
Please see docs.dapr.io for certificate renewal instructions to avoid service interruptions.
```

### 相关链接

- [更新 Kubernetes 集群中的 Dapr]({{< ref kubernetes-upgrade.md >}})