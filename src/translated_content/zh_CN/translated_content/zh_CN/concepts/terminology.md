---
type: docs
title: "Dapr 术语和定义"
linkTitle: "术语"
weight: 900
description: Dapr文档中的通用术语和缩略语的定义
---

本页详细介绍了您在 Dapr 文档中可能遇到的所有常用术语。

| Term               | 定义                                                                                                                       | 详情                                                                                                   |
|:------------------ | ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| App/Application    | A running service/binary, usually one that you as the user create and run.                                               |                                                                                                      |
| Building block     | Dapr 为用户提供的 API，以帮助创建微服务和应用程序。                                                                                           | [Dapr 构建块]({{< ref building-blocks-concept.md >}})                                                   |
| Component          | 由 Dapr 构建块使用的模块化功能类型，可以单独使用，也可以与其他组件的集合一起使用。                                                                             | [Dapr 组件]({{< ref components-concept.md >}})                                                         |
| Configuration      | 一个 YAML 文件，声明 Dapr sidecars 或 Dapr 控制面板的所有设置。 您可以在这里配置控制平面 mTLS 设置，或应用程序实例的跟踪和中间件设置。                                     | [Dapr 配置]({{< ref configuration-concept.md >}})                                                      |
| Dapr               | 分布式应用运行时(Distributed Application Runtime 的缩写)                                                                            | [Dapr 概述]({{< ref overview.md >}})                                                                   |
| Dapr control plane | 在托管平台（如 Kubernetes 集群）上作为 Dapr 安装一部分的服务集合。 允许启用 Dapr 的应用程序在平台上运行，并处理 Dapr 功能，如 actor placement 、Dapr sidecar 注入或证书签发/延续。 | [自托管概述]({{< ref self-hosted-overview >}})<br />[Kubernetes概述]({{< ref kubernetes-overview >}}) |
| Self-hosted        | 您可以在 Windows/macOS/Linux 机器用 Dapr 运行您的应用程序。 Dapr 提供在机器上以"自托管"模式运行的能力。                                                    | [自托管模式]({{< ref self-hosted-overview.md >}})                                                         |
| 服务                 | 正在运行的应用程序或二进制文件。 可用于指您的应用程序，或 Dapr 应用程序。                                                                                 |                                                                                                      |
| Sidecar            | 作为单独的进程或容器与应用程序一起运行的程序。                                                                                                  | [Sidecar 模式](https://docs.microsoft.com/azure/architecture/patterns/sidecar)                         |
