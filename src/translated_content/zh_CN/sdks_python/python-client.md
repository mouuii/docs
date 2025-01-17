---
type: docs
title: "开始使用 Dapr 客户端 Python SDK"
linkTitle: "客户端"
weight: 10000
description: 如何使用 Dapr Python SDK 启动和运行
---

Dapr 客户端包允许您从 Python 应用程序中与其他 Dapr 应用程序进行交互。

## 前提

- 安装 [Dapr CLI]({{< ref install-dapr-cli.md >}})
- 初始化[Dapr环境]({{< ref install-dapr-selfhost.md >}})
- 安装[Python 3.7+](https://www.python.org/downloads/)
- 安装[Dapr Python 模块]({{< ref "python#install-the0dapr-module" >}})

## 导入客户端包

Dapr 包包含 `DaprClient` ，该工具包将用于创建和使用客户端。

```python
from dapr.clients import DaprClient
```

## 构建块

Python SDK 允许你与所有的 [Dapr 构建块]({{< ref building-blocks >}}) 进行交互。

### 调用服务

```python 
from dapr.clients import DaprClient

with DaprClient() as d:
    # invoke a method (gRPC or HTTP GET)    
    resp = d.invoke_method('service-to-invoke', 'method-to-invoke', data='{"message":"Hello World"}')

    # for other HTTP verbs the verb must be specified
    # invoke a 'POST' method (HTTP only)    
    resp = d.invoke_method('service-to-invoke', 'method-to-invoke', data='{"id":"100", "FirstName":"Value", "LastName":"Value"}', http_verb='post')
```

- 有关服务调用的完整指南，请访问 [如何：调用服务]({{< ref howto-invoke-discover-services.md >}})。
- 请访问 [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples/invoke-simple) ，了解代码示例和说明，尝试服务调用。

### 保存和获取应用程序状态

```python
from dapr.clients import DaprClient

with DaprClient() as d:
    # Save state
    d.save_state(store_name="statestore", key="key1", value="value1")

    # Get state
    data = d.get_state(store_name="statestore", key="key1").data

    # Delete state
    d.delete_state(store_name="statestore", key="key1")
```

- 有关状态操作的完整列表，请访问 [如何：获取 & 保存 状态。]({{< ref howto-get-save-state.md >}})。
- 请访问 [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples/state_store) ，了解代码示例和说明，以尝试使用状态管理。

### 查询应用程序状态（Alpha）

```python
    from dapr import DaprClient

    query = '''
    {
        "filter": {
            "EQ": { "state": "CA" }
        },
        "sort": [
            {
                "key": "person.id",
                "order": "DESC"
            }
        ]
    }
    '''

    with DaprClient() as d:
        resp = d.query_state(
            store_name='state_store',
            query=query,
            states_metadata={"metakey": "metavalue"},  # optional
        )
```

- For a full list of state store query options visit [How-To: Query state]({{< ref howto-state-query-api.md >}}).
- 请访问 [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples/state_store_query) ，了解代码示例和说明，以尝试使用状态管理。

### 发布 & 订阅消息

##### 发布消息

```python
from dapr.clients import DaprClient

with DaprClient() as d:
    resp = d.publish_event(pubsub_name='pubsub', topic_name='TOPIC_A', data='{"message":"Hello World"}')
```

##### 订阅消息

```python
from cloudevents.sdk.event import v1
from dapr.ext.grpc import App
import json

app = App()

# Default subscription for a topic
@app.subscribe(pubsub_name='pubsub', topic='TOPIC_A')
def mytopic(event: v1.Event) -> None:
    data = json.loads(event.Data())
    print(f'Received: id={data["id"]}, message="{data ["message"]}"' 
          ' content_type="{event.content_type}"',flush=True)

# Specific handler using Pub/Sub routing
@app.subscribe(pubsub_name='pubsub', topic='TOPIC_A',
               rule=Rule("event.type == \"important\"", 1))
def mytopic_important(event: v1.Event) -> None:
    data = json.loads(event.Data())
    print(f'Received: id={data["id"]}, message="{data ["message"]}"' 
          ' content_type="{event.content_type}"',flush=True)
```

- For a full list of state operations visit [How-To: Publish & subscribe]({{< ref howto-publish-subscribe.md >}}).
- 请访问 [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples/pubsub-simple) 以获取代码样本和说明，尝试使用发布和订阅。

### 与输出绑定交互

```python
from dapr.clients import DaprClient

with DaprClient() as d:
    resp = d.invoke_binding(binding_name='kafkaBinding', operation='create', data='{"message":"Hello World"}')
```

- For a full guide on output bindings visit [How-To: Use bindings]({{< ref howto-bindings.md >}}).
- 请访问 [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples/invoke-binding) 以获取代码示例和说明，尝试输出绑定。

### Retrieve secrets

```python
from dapr.clients import DaprClient

with DaprClient() as d:
    resp = d.get_secret(store_name='localsecretstore', key='secretKey')
```

- For a full guide on secrets visit [How-To: Retrieve secrets]({{< ref howto-secrets.md >}}).
- 请访问 [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples/secret_store) 以获取代码示例和说明，以尝试检索秘密。

### 获取配置

```python
from dapr.clients import DaprClient

with DaprClient() as d:
    # Get Configuration
    configuration = d.get_configuration(store_name='configurationstore', keys=['orderId'], config_metadata={})
```

### Subscribe to configuration

```python
import asyncio
from time import sleep
from dapr.clients import DaprClient

async def executeConfiguration():
    with DaprClient() as d:
        storeName = 'configurationstore'

        key = 'orderId'

        # Wait for sidecar to be up within 20 seconds.
        d.wait(20)

        # Subscribe to configuration by key.
        configuration = await d.subscribe_configuration(store_name=storeName, keys=[key], config_metadata={})
        while True:
            if configuration != None:
                items = configuration.get_items()
                for key, item in items:
                    print(f"Subscribe key={key} value={item.value} version={item.version}", flush=True)
            else:
                print("Nothing yet")
        sleep(5)

asyncio.run(executeConfiguration())
```

- For a full list of state operations visit [How-To: Get & save state]({{< ref howto-manage-configuration.md >}}).
- 请访问 [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples/configuration) ，了解代码示例和说明，以尝试使用状态管理。

## 相关链接
- [Python SDK 示例](https://github.com/dapr/python-sdk/tree/master/examples)
