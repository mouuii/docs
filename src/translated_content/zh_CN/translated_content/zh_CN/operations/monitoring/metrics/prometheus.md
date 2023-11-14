---
type: docs
title: "How-To: Observe metrics with Prometheus"
linkTitle: "Prometheus"
weight: 4000
description: "Use Prometheus to collect time-series data relating to the execution of the Dapr runtime itself"
---

## Setup Prometheus Locally
To run Prometheus on your local machine, you can either [install and run it as a process](#install) or run it as a [Docker container](#Run-as-Container).

### Install
{{% alert title="Note" color="warning" %}}
如果你计划以 Docker 容器的形式运行 Prometheus，则无需安装它。 请参考[容器](#run-as-container)说明。
{{% /alert %}}

要安装 Prometheus，请按照[这里](https://prometheus.io/docs/prometheus/latest/getting_started/)概述的适用于你的操作系统的步骤。

### 配置
现在你已经安装了 Prometheus，你需要创建一个配置。

下面是一个 Prometheus 配置的例子，将其保存到一个文件中，即 `/tmp/prometheus.yml` 或 `C：\Temp\prometheus.yml`
```yaml
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'dapr'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090'] # Replace with Dapr metrics port if not default
```

### 作为进程运行
用你的配置运行 Prometheus，以启动它从指定目标收集指标。
```bash
./prometheus --config.file=/tmp/prometheus.yml --web.listen-address=:8080
```
> 我们改变端口，这样就不会与 Dapr 自己的度量端点冲突。

如果你目前没有运行 Dapr 应用程序，则目标将显示为脱机。 为了启动收集指标，你必须用符合配置中提供的目标的指标端口启动 Dapr。

一旦 Prometheus 运行，您将能够通过访问 `http://localhost:8080` 来访问其仪表板。

### 作为容器运行
要在你的本地机器上作为 Docker 容器运行 Prometheus，首先要确保你已经安装和运行 [Docker](https://docs.docker.com/install/)。

然后你可以使用 Docker 容器来运行 Prometheus。
```bash
docker run \
    --net=host \
    -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.listen-address=:8080
```
`--net=host` 确保 Prometheus 实例将能够连接到主机上运行的任何 Dapr 实例。 如果您还计划在容器中运行 Dapr 应用程序，则需要在共享 Docker 网络上运行它们，并使用正确的目标地址更新配置。

一旦 Prometheus 运行，您将能够通过访问 `http://localhost:8080` 来访问其仪表板。

## 在 Kubernetes 上部署 Prometheus

### Prerequisites

- Kubernetes (> 1.14)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm 3](https://helm.sh/)

### 安装 Prometheus

1.  First create namespace that can be used to deploy the Grafana and Prometheus monitoring tools

```bash
kubectl create namespace dapr-monitoring
```

2. 安装 Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install dapr-prom prometheus-community/prometheus -n dapr-monitoring
```

如果您正在使用 minikube 或者想要禁用持久化卷来开发，您可以使用以下命令禁用它：

```bash
helm install dapr-prom prometheus-community/prometheus -n dapr-monitoring
 --set alertmanager.persistentVolume.enable=false --set pushgateway.persistentVolume.enabled=false --set server.persistentVolume.enabled=false
```

3. 校验

确保 Prometheus 正在群集中运行。

```bash
kubectl get pods -n dapr-monitoring

NAME                                                READY   STATUS    RESTARTS   AGE
dapr-prom-kube-state-metrics-9849d6cc6-t94p8        1/1     Running   0          4m58s
dapr-prom-prometheus-alertmanager-749cc46f6-9b5t8   2/2     Running   0          4m58s
dapr-prom-prometheus-node-exporter-5jh8p            1/1     Running   0          4m58s
dapr-prom-prometheus-node-exporter-88gbg            1/1     Running   0          4m58s
dapr-prom-prometheus-node-exporter-bjp9f            1/1     Running   0          4m58s
dapr-prom-prometheus-pushgateway-688665d597-h4xx2   1/1     Running   0          4m58s
dapr-prom-prometheus-server-694fd8d7c-q5d59         2/2     Running   0          4m58s
```

## 示例

<div class="embed-responsive embed-responsive-16by9">
    <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/8W-iBDNvCUM?start=2577" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

## 参考

* [安装 Prometheus](https://github.com/prometheus-community/helm-charts)
* [Prometheus 查询语言](https://prometheus.io/docs/prometheus/latest/querying/basics/)