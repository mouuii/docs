---
type: docs
title: "Apple Push Notification Service绑定规范"
linkTitle: "Apple Push Notification Service"
description: "有关 Apple 推送通知服务绑定组件的详细文档"
aliases:
  - "/zh-hans/operations/components/setup-bindings/supported-bindings/apns/"
---

## Component format

To setup Apple Push Notifications binding create a component of type `bindings.apns`. See [this guide]({{< ref "howto-bindings.md#1-create-a-binding" >}}) on how to create and apply a binding configuration.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <NAME>
spec:
  type: bindings.apns
  version: v1
  metadata:
    - name: development
      value: <bool>
    - name: key-id
      value: <APPLE_KEY_ID>
    - name: team-id
      value: <APPLE_TEAM_ID>
    - name: private-key
      secretKeyRef:
        name: <SECRET>
        key: <SECRET-KEY-NAME>
```
## 元数据字段规范

| Field       | 必填 | 绑定支持   | 详情                                                                                                                                                        | 示例                 |
| ----------- |:--:| ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------ |
| development | 是  | Output | Tells the binding which APNs service to use. Set to `"true"` to use the development service or `"false"` to use the production service. Default: `"true"` | `"true"`           |
| key-id      | 是  | 输出     | 来自 Apple 开发者门户的私钥的标识符。                                                                                                                                    | `"private-key-id`" |
| team-id     | 是  | 输出     | 来自 Apple 开发者门户的组织或作者的标识符。                                                                                                                                 | `"team-id"`        |
| private-key | 是  | 输出     | 是一个PKCS #8格式的私钥。 其目的是将私钥存储在密钥存储中，而不是直接暴露在配置中。 请参阅[这里](#private-key)了解更多详情。                                                                                | `"pem file"`       |

### Private key
APNS绑定需要一个加密私钥，以便为APNS服务生成认证令牌。 私钥可以从Apple开发者门户生成，并以PKCS #8文件的形式提供，私钥以PEM格式存储。 私钥应该存储在Dapr的密钥存储中，而不是直接存储在绑定的配置文件中。

APNS绑定的配置文件示例如下所示：
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: apns
spec:
  type: bindings.apns
  metadata:
  - name: development
    value: false
  - name: key-id
    value: PUT-KEY-ID-HERE
  - name: team-id
    value: PUT-APPLE-TEAM-ID-HERE
  - name: private-key
    secretKeyRef:
      name: apns-secrets
      key: private-key
```
如果使用Kubernetes，一个示例的密钥配置可能是这样的：
```yaml
apiVersion: v1
kind: Secret
metadata:
    name: apns-secrets
stringData:
    private-key: |
        -----BEGIN PRIVATE KEY-----
        KEY-DATA-GOES-HERE
        -----END PRIVATE KEY-----
```

## 绑定支持

该组件支持如下操作的 **输出绑定** ：

- `create`

## 推送通知格式

APNS 绑定是Apple Push Notification Service的通行证封装。 APNS绑定会直接将请求发送到APNS服务，不需要任何翻译。 因此，了解APNS服务所期望的推送通知的有效载荷非常重要。 有效载荷格式在[这里](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification)有详细文档。

### 请求格式

```json
{
    "data": {
        "aps": {
            "alert": {
                "title": "New Updates!",
                "body": "There are new updates for your review"
            }
        }
    },
    "metadata": {
        "device-token": "PUT-DEVICE-TOKEN-HERE",
        "apns-push-type": "alert",
        "apns-priority": "10",
        "apns-topic": "com.example.helloworld"
    },
    "operation": "create"
}
```
<!-- IGNORE_LINKS -->
`data`对象包含完整的推送通知规范，如[Apple文档](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification)中所述。 `data`对象将直接发送到APNs服务。

除了`device-token`值之外，[Apple文档](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns)中指定的HTTP头信息可以作为元数据字段发送，并将包含在向APNs服务发出的HTTP请求中。
<!-- END_IGNORE -->

### 响应格式

```json
{
    "messageID": "UNIQUE-ID-FOR-NOTIFICATION"
}
```

## 相关链接

- [Basic schema for a Dapr component]({{< ref component-schema >}})
- [绑定构建块]({{< ref bindings >}})
- [如何通过 input binding 触发应用]({{< ref howto-triggers.md >}})
- [How-To：使用绑定与外部资源进行交互]({{< ref howto-bindings.md >}})
- [Bindings API 引用]({{< ref bindings_api.md >}})
