---
type: docs
title: "Developing Dapr applications with remote dev containers"
linkTitle: "Remote dev containers"
weight: 50000
description: "How to setup a remote dev container environment with Dapr"
---

The Visual Studio Code [Remote Containers extension](https://code.visualstudio.com/docs/remote/containers) lets you use a Docker container as a full-featured development environment without installing any additional frameworks or packages to your local filesystem.

Dapr 为 NodeJS 和 C# 预先构建了 Docker 远程容器。 您可以选择一个现成的环境。 请注意，这些预构建的容器会自动更新到最新的 Dapr 版本。

### 设置远程开发容器

#### Prerequisites
<!-- IGNORE_LINKS -->
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
<!-- END_IGNORE -->
- [Visual Studio Code](https://code.visualstudio.com/)
- [VSCode 远程开发扩展包](https://aka.ms/vscode-remote/download/extension)

#### 创建远程 Dapr 容器
1. Open your application workspace in VS Code
2. 在 command palette 中 (CTRL+SHIFT+P) 输入并选择 `Remote-Containers: Add Development Container Configuration Files...` <br /><img src="/images/vscode-remotecontainers-addcontainer.png" alt="添加远程容器的截图" width="700" />
3. 输入 `dapr` 来过滤列表到可用的 Dapr 远程容器，并选择符合您应用程序的语言容器。 请注意，您可能需要选择 `Show All Definitions...` <br /><img src="/images/vscode-remotecontainers-daprcontainers.png" alt="添加 dapr 容器的截图" width="700" />
4. 按照提示在容器中重新编译您的应用程序。 <br /><img src="/images/vscode-remotecontainers-reopen.png" alt="在开发容器中重新打开应用程序的截图" width="700" />

#### 示例
观看有关如何使用应用程序的 Dapr VS 代码远程容器的 [视频](https://www.bilibili.com/video/BV1QK4y1p7fn?p=8&t=120)。

<div class="embed-responsive embed-responsive-16by9">
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/D2dO4aGpHcg?start=120" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>