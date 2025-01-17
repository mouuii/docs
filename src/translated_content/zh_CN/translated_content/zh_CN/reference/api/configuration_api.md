---
type: docs
title: "Configuration API reference"
linkTitle: "Configuration API"
description: "Detailed documentation on the configuration API"
weight: 100
---

## Get Configuration

This endpoint lets you get configuration from a store.

### HTTP Request

```
GET http://localhost:<daprPort>/v1.0-alpha1/configuration/<storename>
```

#### URL 参数

| Parameter   | 说明                                                                                                        |
| ----------- | --------------------------------------------------------------------------------------------------------- |
| `daprPort`  | The Dapr port                                                                                             |
| `storename` | The `metadata.name` field component file. Refer to the [component schema]({{< ref component-schema.md>}}) |

#### 查询参数

If no query parameters are provided, all configuration items are returned. To specify the keys of the configuration items to get, use one or more `key` query parameters. For example:

```
GET http://localhost:<daprPort>/v1.0-alpha1/configuration/mystore?key=config1&key=config2
```

To retrieve all configuration items:

```
GET http://localhost:<daprPort>/v1.0-alpha1/configuration/mystore
```

#### 请求正文

None

### HTTP 响应

#### 响应码

| Code  | 说明                                                                   |
| ----- | -------------------------------------------------------------------- |
| `204` | Get operation successful                                             |
| `400` | Configuration store is missing or misconfigured or malformed request |
| `500` | Failed to get configuration                                          |

#### Response Body

JSON-encoded value of key/value pairs for each configuration item.

### 示例

```shell
curl -X GET 'http://localhost:3500/v1.0-alpha1/configuration/mystore?key=myConfigKey' 
```

> The above command returns the following JSON:

```json
[{"key":"myConfigKey","value":"myConfigValue"}]
```

## Subscribe Configuration

This endpoint lets you subscribe to configuration changes. Notifications happen when values are updated or deleted in the configuration store. This enables the application to react to configuration changes.

### HTTP 请求

```
GET http://localhost:<daprPort>/v1.0-alpha1/configuration/<storename>/subscribe
```

#### URL 参数

| Parameter   | 说明                                                                                                        |
| ----------- | --------------------------------------------------------------------------------------------------------- |
| `daprPort`  | The Dapr port                                                                                             |
| `storename` | The `metadata.name` field component file. Refer to the [component schema]({{< ref component-schema.md>}}) |

#### 查询参数

If no query parameters are provided, all configuration items are subscribed to. To specify the keys of the configuration items to subscribe to, use one or more `key` query parameters. For example:

```
GET http://localhost:<daprPort>/v1.0-alpha1/configuration/mystore/subscribe?key=config1&key=config2
```

To subscribe to all changes:

```
GET http://localhost:<daprPort>/v1.0-alpha1/configuration/mystore/subscribe
```

#### 请求正文

None

### HTTP 响应

#### 响应码

| Code  | 说明                                                                   |
| ----- | -------------------------------------------------------------------- |
| `200` | Subscribe operation successful                                       |
| `400` | Configuration store is missing or misconfigured or malformed request |
| `500` | Failed to subscribe to configuration changes                         |

#### Response Body

JSON-encoded value

### 示例

```shell
curl -X GET 'http://localhost:3500/v1.0-alpha1/configuration/mystore/subscribe?key=myConfigKey' 
```

> The above command returns the following JSON:

```json
{
  "id": "<unique-id>"
}
```

The returned `id` parameter can be used to unsubscribe to the specific set of keys provided on the subscribe API call. This should be retained by the application.

## Unsubscribe Configuration

This endpoint lets you unsubscribe to configuration changes.

### HTTP Request

```
GET http://localhost:<daprPort>/v1.0-alpha1/configuration/<storename>/<subscription-id>/unsubscribe
```

#### URL 参数

| Parameter         | 说明                                                                                                        |
| ----------------- | --------------------------------------------------------------------------------------------------------- |
| `daprPort`        | The Dapr port                                                                                             |
| `storename`       | The `metadata.name` field component file. Refer to the [component schema]({{< ref component-schema.md>}}) |
| `subscription-id` | The value from the `id` field returned from the response of the subscribe endpoint                        |

#### 查询参数

None

#### 请求正文

None

### HTTP 响应

#### 响应码

| Code  | 说明                                                                   |
| ----- | -------------------------------------------------------------------- |
| `200` | Unsubscribe operation successful                                     |
| `400` | Configuration store is missing or misconfigured or malformed request |
| `500` | Failed to unsubscribe to configuration changes                       |

#### 响应正文

```
{
    "ok" : true
}
```

### 示例

```shell
curl -X GET 'http://localhost:3500/v1.0-alpha1/configuration/mystore/bf3aa454-312d-403c-af95-6dec65058fa2/unsubscribe' 
```

## Optional application (user code) routes

### Provide a route for Dapr to send configuration changes

subscribing to configuration changes, Dapr invokes the application whenever a configuration item changes. Your application can have a `/configuration` endpoint that is called for all key updates that are subscribed to. The endpoint(s) can be made more specific for a given configuration store by adding `/<store-name>` and for a specific key by adding `/<store-name>/<key>` to the route.

#### HTTP Request

```
POST http://localhost:<appPort>/configuration/<store-name>/<key>
```

#### URL 参数

| Parameter   | 说明                                                                                                        |
| ----------- | --------------------------------------------------------------------------------------------------------- |
| `appPort`   | The application port                                                                                      |
| `storename` | The `metadata.name` field component file. Refer to the [component schema]({{< ref component-schema.md>}}) |
| `key`       | The key subscribed to                                                                                     |

#### 请求正文

A list of configuration items for a given subscription id. Configuration items can have a version associated with them, which is returned in the notification.

```json
{
    "id": "<subscription-id>",
    "items": [
        "key": "<key-of-configuration-item>",
        "value": "<new-value>",
        "version": "<version-of-item>"
    ]
}
```

#### 示例

```json
{
    "id": "bf3aa454-312d-403c-af95-6dec65058fa2",
    "items": [
        "key": "config-1",
        "value": "abcdefgh",
        "version": "1.1"
    ]
}
```


## Next Steps

- [Configuration API overview]({{< ref configuration-api-overview.md >}})
- [How-To: Manage configuration from a store]({{< ref howto-manage-configuration.md >}})
