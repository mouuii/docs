---
type: docs
title: "status CLI 命令参考"
linkTitle: "status"
description: "有关 status CLI 命令的详细信息"
---

### 说明

显示 Dapr 服务的健康状况。

### 支持的平台

- [Kubernetes]({{< ref kubernetes >}})

### 用法

```bash
dapr status -k
```

### 参数

| Name                 | 环境变量 | 默认值     | 说明                             |
| -------------------- | ---- | ------- | ------------------------------ |
| `--help`, `-h`       |      |         | 显示此帮助消息                        |
| `--kubernetes`, `-k` |      | `false` | 显示 Kubernetes 集群上 Dapr 服务的运行状况 |

### 示例

```bash
# 从Kubernetes集群中获取Dapr服务的状态
dapr status -k
```

### 警告信息
此命令可以发出警告消息。

#### 根证书续订警告
如果部署到 Kubernetes 集群的 mtls 根证书在 30 天内过期，则会显示以下警告消息：

```
Dapr root certificate of your Kubernetes cluster expires in <n> days. Expiry date: <date:time> UTC. 
Please see docs.dapr.io for certificate renewal instructions to avoid service interruptions.
```