---
type: docs
title: "使用 token 认证来自 Dapr 的请求"
linkTitle: "应用 API 令牌身份验证"
weight: 4000
description: "要求来自 Dapr 的每个传入 API 请求都包含身份验证令牌"
---

对于某些构建块，例如发布/订阅、服务调用和输入绑定，Dapr 通过 HTTP 或 gRPC 与应用进行通信。 要使应用程序能够对从 Dapr sidecar 发出的请求进行身份验证，您可以将 Dapr 配置为将 API token 作为标头（在 HTTP 请求中）或元数据（在 gRPC 请求中）发送。

## 创建令牌

Dapr 使用共享令牌进行 API 身份验证。 您可以自由定义要使用的 API 令牌。

Although Dapr does not impose any format for the shared token, a good idea is to generate a random byte sequence and encode it to Base64. For example, this command generates a random 32-byte key and encodes that as Base64:

```sh
openssl rand 16 | base64
```

## 在 Dapr 中配置应用 API 令牌身份验证

令牌认证配置在 Kubernetes 和 自托管 Dapr deployments 下稍有不同：

### 自托管

在自托管场景中， Dapr 查找是否存在 `APP_API_TOKEN` 环境变量。 If that environment variable is set when the `daprd` process launches, Dapr includes the token when calling an app:

```shell
export APP_API_TOKEN=<token>
```

To rotate the configured token, update the `APP_API_TOKEN` environment variable to the new value and restart the `daprd` process.

### Kubernetes

In a Kubernetes deployment, Dapr leverages Kubernetes secrets store to hold the shared token. To start, create a new secret:

```shell
kubectl create secret generic app-api-token --from-literal=token=<token>
```

> 注意，上述 secret 需要在你希望开启 Dapr token 认证的命名空间中创建

若要指示 Dapr 在向应用发送请求时使用秘密中的令牌，请向 deployment 模板规范添加注解：

```yaml
annotations:
  dapr.io/enabled: "true"
  dapr.io/app-token-secret: "app-api-token" # name of the Kubernetes secret
```

当 Deployment 部署后，Dapr sidecar 注入器会自动创建一个秘密，并将实际值注入到 `APP_API_TOKEN` 环境变量中。

## 更新令牌

### 自托管

To rotate the configured token in self-hosted, update the `APP_API_TOKEN` environment variable to the new value and restart the `daprd` process.

### Kubernetes

To rotate the configured token in Kubernates, update the previously-created secret with the new token in each namespace. You can do that using `kubectl patch` command, but a simpler way to update these in each namespace is by using a manifest:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-api-token
type: Opaque
data:
  token: <your-new-token>
```

然后将其 apply 到每个命名空间：

```shell
kubectl apply --file token-secret.yaml --namespace <namespace-name>
```

为了让 Dapr 开始使用新令牌，需要对你的每个 deployment 进行滚动升级：

```shell
kubectl rollout restart deployment/<deployment-name> --namespace <namespace-name>
```

> Assuming your service is configured with more than one replica, the key rotation process does not result in any downtime.

## 验证来自 Dapr 的请求

Once app token authentication is configured in Dapr, all requests *coming from Dapr* include the token.

### HTTP

In case of HTTP, in your code look for the HTTP header `dapr-api-token` in incoming requests:

```text
dapr-api-token: <token>
```

### gRPC

当使用 gRPC 协议时，请检查入站 gRPC 请求的元数据（metadata）上的 API 令牌 ：

```text
dapr-api-token[0].
```

## 从应用程序访问令牌

### Kubernetes

在 Kubernetes 中，建议将秘密作为环境变量挂载到 pod 中。 假定我们创建了一个名为 `app-api-token` 的秘密来保存令牌：

```yaml
containers:
  - name: mycontainer
    image: myregistry/myapp
    envFrom:
    - secretRef:
      name: app-api-token
```

### 自托管

在自托管模式下，您可以将令牌设置为应用程序的环境变量 ：

```sh
export APP_API_TOKEN=<my-app-token>
```

## 相关链接

- 了解 [Dapr 安全概念]({{< ref security-concept.md >}})
- 了解[如何在 Dapr 中启用 API 令牌身份验证]({{< ref api-token.md >}})
