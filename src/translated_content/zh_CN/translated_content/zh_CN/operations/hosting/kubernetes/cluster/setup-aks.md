---
type: docs
title: "创建 Azure Kubernetes 服务集群"
linkTitle: "Azure Kubernetes Service （AKS）"
weight: 2000
description: >
  如何在 Azure Kubernetes 集群上设置 Dapr。
---

# 设置 Azure Kubernetes 服务集群

## Prerequisites

- [Docker](https://docs.docker.com/install/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)

## 部署 Azure Kubernetes 服务集群

This guide walks you through installing an Azure Kubernetes Service cluster. If you need more information, refer to [Quickstart: Deploy an Azure Kubernetes Service (AKS) cluster using the Azure CLI](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough)

1. Login to Azure

```bash
az login
```

2. 设置默认订阅

```bash
az account set -s [your_subscription_id]
```

3. 创建资源组

```bash
az group create --name [your_resource_group] --location [region]
```

4. 创建 Azure Kubernetes Service 集群

> **注意：** 要使用特定版本的 Kubernetes 请使用 `--kubernetes-version` (1.13.x 或需要更新版本)

```bash
az aks create --resource-group [your_resource_group] --name [your_aks_cluster_name] --node-count 2 --enable-addons http_application_routing --generate-ssh-keys
```

5. 获取 Azure Kubernetes 集群的访问凭据

```bash
az aks get-credentials -n [your_aks_cluster_name] -g [your_resource_group]
```

## 下一步

{{< button text="Install Dapr using the AKS Dapr extension >>" page="azure-kubernetes-service-extension" >}}
