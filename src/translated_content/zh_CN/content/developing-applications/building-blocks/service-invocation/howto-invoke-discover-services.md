---
type: docs
title: "入门指南：发现并调用服务"
linkTitle: "操作方法：使用 HTTP 调用"
description: "入门指南指导如何使用 Dapr 服务在分布式应用程序中调用其它服务"
weight: 2000
---

This article demonstrates how to deploy services each with an unique application ID for other services to discover and call endpoints on them using service invocation over HTTP.

<img src="/images/building-block-service-invocation-example.png" width=1000 height=500 alt="显示示例服务的服务调用的图示">

{{% alert title="Note" color="primary" %}}
 If you haven't already, [try out the service invocation quickstart]({{< ref serviceinvocation-quickstart.md >}}) for a quick walk-through on how to use the service invocation API.

{{% /alert %}}

## Choose an ID for your service

Dapr 允许您为您的应用分配一个全局唯一ID。 此 ID 为您的应用程序封装了状态，不管它可能有多少实例。

{{< tabs Dotnet Java Python Go Javascript Kubernetes>}}

{{% codetab %}}

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 dotnet run

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 dotnet run

```

如果您的应用使用 SSL 连接，您可以告诉Dapr 在不安全的 SSL 连接中调用您的应用：

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 --app-ssl dotnet run

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 --app-ssl dotnet run

```

{{% /codetab %}}

{{% codetab %}}

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 mvn spring-boot:run

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 mvn spring-boot:run

```

如果您的应用使用 SSL 连接，您可以告诉Dapr 在不安全的 SSL 连接中调用您的应用：

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 --app-ssl mvn spring-boot:run

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 --app-ssl mvn spring-boot:run

```

{{% /codetab %}}

{{% codetab %}}

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 -- python3 CheckoutService.py

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 -- python3 OrderProcessingService.py

```

如果您的应用使用 SSL 连接，您可以告诉Dapr 在不安全的 SSL 连接中调用您的应用：

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 --app-ssl -- python3 CheckoutService.py

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 --app-ssl -- python3 OrderProcessingService.py

```

{{% /codetab %}}

{{% codetab %}}

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 go run CheckoutService.go

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 go run OrderProcessingService.go

```

如果您的应用使用 SSL 连接，您可以告诉Dapr 在不安全的 SSL 连接中调用您的应用：

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 --app-ssl go run CheckoutService.go

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 --app-ssl go run OrderProcessingService.go

```

{{% /codetab %}}

{{% codetab %}}

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 npm start

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 npm start

```

如果您的应用使用 SSL 连接，您可以告诉Dapr 在不安全的 SSL 连接中调用您的应用：

```bash

dapr run --app-id checkout --app-port 6002 --dapr-http-port 3602 --dapr-grpc-port 60002 --app-ssl npm start

dapr run --app-id orderprocessing --app-port 6001 --dapr-http-port 3601 --dapr-grpc-port 60001 --app-ssl npm start

```

{{% /codetab %}}

{{% codetab %}}

### 在部署到Kubernetes时设置一个应用程序的ID

在 Kubernetes 中，在您的pod 上设置 `dapr.io/app-id` 注解：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <language>-app
  namespace: default
  labels:
    app: <language>-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <language>-app
  template:
    metadata:
      labels:
        app: <language>-app
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "orderprocessingservice"
        dapr.io/app-port: "6001"
...
```

*如果应用程序使用 SSL 连接，那么可以使用 `app-ssl: "true"` 注解 (完整列表 [此处]({{< ref arguments-annotations-overview.md >}})) 告知 Dapr 在不安全的 SSL 连接上调用应用程序。*

{{% /codetab %}}

{{< /tabs >}}

## Invoke the service

要使用 Dapr 调用应用程序，您可以在任意 Dapr 实例中使用 `调用` API。 Sidecar 编程模型鼓励每个应用程序与自己的 Dapr 实例对话。 Dapr 边车实例会相互发现并进行通信。

下面是利用 Dapr SDK 进行服务调用的代码示例。

{{< tabs Dotnet Java Python Go Javascript>}}

{{% codetab %}}

```csharp
//dependencies
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Threading.Tasks;
using Dapr.Client;
using Microsoft.AspNetCore.Mvc;
using System.Threading;

//code
namespace EventService
{
    class Program
    {
        static async Task Main(string[] args)
        {
           while(true) {
                System.Threading.Thread.Sleep(5000);
                Random random = new Random();
                int orderId = random.Next(1,1000);
                CancellationTokenSource source = new CancellationTokenSource();
                CancellationToken cancellationToken = source.Token;
                using var client = new DaprClientBuilder().Build();
                //Using Dapr SDK to invoke a method
                var result = client.CreateInvokeMethodRequest(HttpMethod.Get, "checkout", "checkout/" + orderId, cancellationToken);
                await client.InvokeMethodAsync(result);
                Console.WriteLine("Order requested: " + orderId);
                Console.WriteLine("Result: " + result);
            }
        }
    }
}
```

{{% /codetab %}}

{{% codetab %}}

```java
//dependencies
import io.dapr.client.DaprClient;
import io.dapr.client.DaprClientBuilder;
import io.dapr.client.domain.HttpExtension;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.Random;
import java.util.concurrent.TimeUnit;

//code
@SpringBootApplication
public class OrderProcessingServiceApplication {

    private static final Logger log = LoggerFactory.getLogger(OrderProcessingServiceApplication.class);

    public static void main(String[] args) throws InterruptedException{
        while(true) {
            TimeUnit.MILLISECONDS.sleep(5000);
            Random random = new Random();
            int orderId = random.nextInt(1000-1) + 1;
            DaprClient daprClient = new DaprClientBuilder().build();
            //Using Dapr SDK to invoke a method
            var result = daprClient.invokeMethod(
                    "checkout",
                    "checkout/" + orderId,
                    null,
                    HttpExtension.GET,
                    String.class
            );
            log.info("Order requested: " + orderId);
            log.info("Result: " + result);
        }
    }
}
```

{{% /codetab %}}

{{% codetab %}}

```python
#dependencies
import random
from time import sleep
import logging
from dapr.clients import DaprClient

#code
logging.basicConfig(level = logging.INFO) 
while True:
    sleep(random.randrange(50, 5000) / 1000)
    orderId = random.randint(1, 1000)
    with DaprClient() as daprClient:
        #Using Dapr SDK to invoke a method
        result = daprClient.invoke_method(
            "checkout",
               f"checkout/{orderId}",
               data=b'',
               http_verb="GET"
        )    
    logging.basicConfig(level = logging.INFO)
    logging.info('Order requested: ' + str(orderId))
    logging.info('Result: ' + str(result))
```

{{% /codetab %}}

{{% codetab %}}

```go
//dependencies
import (
    "context"
    "log"
    "math/rand"
    "time"
    "strconv"
    dapr "github.com/dapr/go-sdk/client"

)

//code
type Order struct {
    orderName string
    orderNum  string
}

func main() {
    for i := 0; i < 10; i++ {
        time.Sleep(5000)
        orderId := rand.Intn(1000-1) + 1
        client, err := dapr.NewClient()
        if err != nil {
            panic(err)
        }
        defer client.Close()
        ctx := context.Background()
        //Using Dapr SDK to invoke a method
        result, err := client.InvokeMethod(ctx, "checkout", "checkout/" + strconv.Itoa(orderId), "get")
        log.Println("Order requested: " + strconv.Itoa(orderId))
        log.Println("Result: ")
        log.Println(result)
    }
}
```

{{% /codetab %}}

{{% codetab %}}

```javascript
//dependencies
import { DaprClient, HttpMethod, CommunicationProtocolEnum } from 'dapr-client'; 

//code
const daprHost = "127.0.0.1"; 

var main = function() {
    for(var i=0;i<10;i++) {
        sleep(5000);
        var orderId = Math.floor(Math.random() * (1000 - 1) + 1);
        start(orderId).catch((e) => {
            console.error(e);
            process.exit(1);
        });
    }
}

async function start(orderId) {
    const client = new DaprClient(daprHost, process.env.DAPR_HTTP_PORT, CommunicationProtocolEnum.HTTP);
    //Using Dapr SDK to invoke a method
    const result = await client.invoker.invoke('checkoutservice' , "checkout/" + orderId , HttpMethod.GET);
    console.log("Order requested: " + orderId);
    console.log("Result: " + result);
}

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

main();
```

{{% /codetab %}}

{{< /tabs >}}

### 其他 URL 格式

要调用 "GET" 端点:

```bash
curl http://localhost:3500/v1.0/invoke/cart/method/add
```

To avoid changing URL paths as much as possible, Dapr provides the following ways to call the service invocation API:

1. 将 URL 中的地址改为 `localhost:<dapr-http-port>`。
2. 添加一个 `dapr-app-id` 头来指定目标服务的ID，或者通过HTTP Basic Auth传递ID。 `http://dapr-app-id。<service-id>@localhost:3602/path`。

For example, the following command:

```bash
curl http://localhost:3500/v1.0/invoke/cart/method/add
```

等同于：

```bash
curl -H 'dapr-app-id: checkout' 'http://localhost:3602/checkout/100' -X POST
```

或：

```bash
curl -H 'dapr-app-id: checkout' 'http://localhost:3602/checkout/100' -X POST
```

使用 CLI：

```bash
dapr invoke --app-id checkout --method checkout/100
```

### 命名空间

当运行于[支持命名空间]({{< ref "service_invocation_api.md#namespace-supported-platforms" >}})的平台时，在您的 app ID 中包含命名空间：`myApp.production`

For example, invoking the example service with a namespace would look like:

```bash
curl http://localhost:3602/v1.0/invoke/checkout.production/method/checkout/100 -X POST
```

有关名称空间的更多信息，请参阅 [跨命名空间 API]({{< ref "service_invocation_api.md#cross-namespace-invocation" >}}) 。

## View traces and logs

Our example above showed you how to directly invoke a different service running locally or in Kubernetes. Dapr:

- Outputs metrics, tracing, and logging information,
- Allows you to visualize a call graph between services and log errors, and
- Optionally, log the payload body.

For more information on tracing and logs, see the [observability]({{< ref observability-concept.md >}}) article.

## 相关链接

- [服务调用概述]({{< ref service-invocation-overview.md >}})
- [服务调用 API 规范]({{< ref service_invocation_api.md >}})