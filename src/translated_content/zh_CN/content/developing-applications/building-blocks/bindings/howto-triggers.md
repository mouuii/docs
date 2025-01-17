---
type: docs
title: "How-To: 使用输入绑定来触发应用程序"
linkTitle: "How-To: 输入绑定"
description: "使用 Dapr 输入绑定来触发由事件驱动的程序"
weight: 200
---

With input bindings, you can trigger your application when an event from an external resource occurs. An external resource could be a queue, messaging pipeline, cloud-service, filesystem, etc. 可选的有效负载和元数据可以与请求一起发送。

Input bindings are ideal for event-driven processing, data pipelines, or generally reacting to events and performing further processing. Dapr input bindings allow you to:

- 接收不包含特定 SDK 或库的事件
- 在不更改代码的情况下替换绑定
- 关注业务逻辑而不是事件资源实现

<img src="/images/howto-triggers/kafka-input-binding.png" width=1000 alt="显示示例服务绑定的图示">

This guide uses a Kafka binding as an example. You can find your preferred binding spec from [the list of bindings components]({{< ref setup-bindings >}}). In this guide:

1. The example invokes the `/binding` endpoint with `checkout`, the name of the binding to invoke.
1. 有效载荷位于必需的 `data` 字段中，并且可以是任何 JSON 可序列化的值。
1. The `operation` field tells the binding what action it needs to take. For example, [the Kafka binding supports the `create` operation]({{< ref "kafka.md#binding-support" >}}).
   - You can check [which operations (specific to each component) are supported for every output binding]({{< ref supported-bindings >}}).

{{% alert title="Note" color="primary" %}}
 If you haven't already, [try out the bindings quickstart]({{< ref bindings-quickstart.md >}}) for a quick walk-through on how to use the bindings API.

{{% /alert %}}

## 创建绑定

Create a `binding.yaml` file and save to a `components` sub-folder in your application directory.

Create a new binding component named `checkout`. Within the `metadata` section, configure the following Kafka-related properties:

- The topic to which you'll publish the message
- The broker

{{< tabs "Self-Hosted (CLI)" Kubernetes >}}

{{% codetab %}}

Use the `--components-path` flag with the `dapr run` command to point to your custom components directory.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: checkout
spec:
  type: bindings.kafka
  version: v1
  metadata:
  # Kafka broker connection setting
  - name: brokers
    value: localhost:9092
  # consumer configuration: topic and consumer group
  - name: topics
    value: sample
  - name: consumerGroup
    value: group1
  # publisher configuration: topic
  - name: publishTopic
    value: sample
  - name: authRequired
    value: "false"
```

{{% /codetab %}}

{{% codetab %}}

To deploy into a Kubernetes cluster, run `kubectl apply -f binding.yaml`.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: checkout
spec:
  type: bindings.kafka
  version: v1
  metadata:
  # Kafka broker connection setting
  - name: brokers
    value: localhost:9092
  # consumer configuration: topic and consumer group
  - name: topics
    value: sample
  - name: consumerGroup
    value: group1
  # publisher configuration: topic
  - name: publishTopic
    value: sample
  - name: authRequired
    value: "false"
```

{{% /codetab %}}

{{< /tabs >}}

## 监听传入事件（输入绑定）

Configure your application to receive incoming events. If you're using HTTP, you need to:
- Listen on a `POST` endpoint with the name of the binding, as specified in `metadata.name` in the `binding.yaml` file.
- Verify your application allows Dapr to make an `OPTIONS` request for this endpoint.

下面是利用 Dapr SDK 演示输出绑定的代码示例。

{{< tabs Dotnet Java Python Go Javascript>}}

{{% codetab %}}

```csharp
//dependencies
using System.Collections.Generic;
using System.Threading.Tasks;
using System;
using Microsoft.AspNetCore.Mvc;

//code
namespace CheckoutService.controller
{
    [ApiController]
    public class CheckoutServiceController : Controller
    {
        [HttpPost("/checkout")]
        public ActionResult<string> getCheckout([FromBody] int orderId)
        {
            Console.WriteLine("Received Message: " + orderId);
            return "CID" + orderId;
        }
    }
}

```

{{% /codetab %}}

{{% codetab %}}

```java
//dependencies
import org.springframework.web.bind.annotation.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import reactor.core.publisher.Mono;

//code
@RestController
@RequestMapping("/")
public class CheckoutServiceController {
    private static final Logger log = LoggerFactory.getLogger(CheckoutServiceController.class);
        @PostMapping(path = "/checkout")
        public Mono<String> getCheckout(@RequestBody(required = false) byte[] body) {
            return Mono.fromRunnable(() ->
                    log.info("Received Message: " + new String(body)));
        }
}

```

{{% /codetab %}}

{{% codetab %}}

```python
#dependencies
import logging
from dapr.ext.grpc import App, BindingRequest

#code
app = App()

@app.binding('checkout')
def getCheckout(request: BindingRequest):
    logging.basicConfig(level = logging.INFO)
    logging.info('Received Message : ' + request.text())

app.run(6002)

```

{{% /codetab %}}

{{% codetab %}}

```go
//dependencies
import (
    "encoding/json"
    "log"
    "net/http"
    "github.com/gorilla/mux"
)

//code
func getCheckout(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    var orderId int
    err := json.NewDecoder(r.Body).Decode(&orderId)
    log.Println("Received Message: ", orderId)
    if err != nil {
        log.Printf("error parsing checkout input binding payload: %s", err)
        w.WriteHeader(http.StatusOK)
        return
    }
}

func main() {
    r := mux.NewRouter()
    r.HandleFunc("/checkout", getCheckout).Methods("POST", "OPTIONS")
    http.ListenAndServe(":6002", r)
}

```

{{% /codetab %}}

{{% codetab %}}

```javascript
//dependencies 
import { DaprServer, CommunicationProtocolEnum } from 'dapr-client'; 

//code
const daprHost = "127.0.0.1"; 
const serverHost = "127.0.0.1";
const serverPort = "6002"; 
const daprPort = "3602"; 

start().catch((e) => {
    console.error(e);
    process.exit(1);
});

async function start() {
    const server = new DaprServer(serverHost, serverPort, daprHost, daprPort, CommunicationProtocolEnum.HTTP);
    await server.binding.receive('checkout', async (orderId) => console.log(`Received Message: ${JSON.stringify(orderId)}`));
    await server.startServer();
}

```

{{% /codetab %}}

{{< /tabs >}}

### 确认事件

Tell Dapr you've successfully processed an event in your application by returning a `200 OK` response from your HTTP handler.

### 拒绝事件

Tell Dapr the event was not processed correctly in your application and schedule it for redelivery by returning any response other than `200 OK`. 例如， `500 Error`。

### 指定自定义路由

默认情况下，传入事件将发送到与输入绑定的名称对应的 HTTP 端点。 You can override this by setting the following metadata property in `binding.yaml`:

```yaml
name: mybinding
spec:
  type: binding.rabbitmq
  metadata:
  - name: route
    value: /onevent
```

### 事件传递保证

事件传递保证由绑定实现控制。 根据绑定实现，事件传递可以正好一次或至少一次。

## 参考资料

- [绑定构建块]({{< ref bindings >}})
- [绑定 API]({{< ref bindings_api.md >}})
- [Components concept]({{< ref components-concept.md >}})
- [Supported bindings]({{< ref supported-bindings >}})