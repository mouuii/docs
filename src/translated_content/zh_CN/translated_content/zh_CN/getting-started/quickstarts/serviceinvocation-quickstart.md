---
type: docs
title: "快速入门：服务调用"
linkTitle: "服务调用"
weight: 71
description: "开始使用 Dapr 的服务调用构建块"
---

With [Dapr's Service Invocation building block](https://docs.dapr.io/developing-applications/building-blocks/service-invocation), your application can communicate reliably and securely with other applications.

<img src="/images/serviceinvocation-quickstart/service-invocation-overview.png" width=800 alt="Diagram showing the steps of service invocation" style="padding-bottom:25px;">

Dapr 提供了几种服务调用方法，你可以根据你的方案选择这些方法。 在本快速入门中，你将启用 checkout 服务以HTTP 代理调用 order-processor 服务中的方法。

在 [概述文章]({{< ref service-invocation-overview.md >}}) 中了解更多关于 Dapr 的服务调用方法。

在继续快速入门之前，请选择您首选的语言。

{{< tabs "Python" "JavaScript" ".NET" "Java" "Go" >}}
 <!-- Python -->
{{% codetab %}}

### Step 1: Pre-requisites

对于此示例，您将需要：

- [Dapr CLI and initialized environment](https://docs.dapr.io/getting-started).
- [Python 3.7+ installed](https://www.python.org/downloads/).
<!-- IGNORE_LINKS -->
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
<!-- END_IGNORE -->

### 第2步：设置环境

克隆[快速入门存储库中提供的示例](https://github.com/dapr/quickstarts/tree/master/service_invocation)。

```bash
git clone https://github.com/dapr/quickstarts.git
```

### 第3步：运行 `order-processor` 服务

在终端窗口中，从快速入门克隆目录的根目录 导航到 `order-processor` 目录。

```bash
cd service_invocation/python/http/order-processor
```

Install the dependencies and build the application:

```bash
pip3 install -r requirements.txt 
```

与 Dapr sidecar 一起运行 `order-processor` 服务。

```bash
dapr run --app-port 8001 --app-id order-processor --app-protocol http --dapr-http-port 3501 -- python3 app.py
```

> **Note**: Since Python3.exe is not defined in Windows, you may need to use `python app.py` instead of `python3 app.py`.

```py
@app.route('/orders', methods=['POST'])
def getOrder():
    data = request.json
    print('Order received : ' + json.dumps(data), flush=True)
    return json.dumps({'success': True}), 200, {
        'ContentType': 'application/json'}


app.run(port=8001)
```

### 第4步：运行 `checkout` 服务

在新终端窗口中，从快速入门克隆目录的根目录导航到 `checkout` 目录。

```bash
cd service_invocation/python/http/checkout
```

Install the dependencies and build the application:

```bash
pip3 install -r requirements.txt 
```

与 Dapr sidecar 一起运行 `checkout` 服务。

```bash
dapr run --app-id checkout --app-protocol http --dapr-http-port 3500 -- python3 app.py
```

> **Note**: Since Python3.exe is not defined in Windows, you may need to use `python app.py` instead of `python3 app.py`.

在 `checkout` 服务中，您会注意到无需重写您的应用程序代码即可使用 Dapr 的服务调用。 您可以通过简单地添加 `dapr-app-id` 标头来启用服务调用，该标头指定目标服务的 ID。

```python
headers = {'dapr-app-id': 'order-processor'}

result = requests.post(
    url='%s/orders' % (base_url),
    data=json.dumps(order),
    headers=headers
)
```

### Step 5: Use with Multi-App Run

You can run the Dapr applications in this quickstart with the [Multi-App Run template]({{< ref multi-app-dapr-run >}}). Instead of running two separate `dapr run` commands for the `order-processor` and `checkout` applications, run the following command:

```sh
dapr run -f .
```

To stop all applications, run:

```sh
dapr stop -f .
```

### Step 6: View the Service Invocation outputs

Dapr invokes an application on any Dapr instance. In the code, the sidecar programming model encourages each application to talk to its own instance of Dapr. The Dapr instances then discover and communicate with one another.

`checkout` service output:

```
== APP == Order passed: {"orderId": 1}
== APP == Order passed: {"orderId": 2}
== APP == Order passed: {"orderId": 3}
== APP == Order passed: {"orderId": 4}
== APP == Order passed: {"orderId": 5}
== APP == Order passed: {"orderId": 6}
== APP == Order passed: {"orderId": 7}
== APP == Order passed: {"orderId": 8}
== APP == Order passed: {"orderId": 9}
== APP == Order passed: {"orderId": 10}
```

`order-processor` service output:

```
== APP == Order received: {"orderId": 1}
== APP == Order received: {"orderId": 2}
== APP == Order received: {"orderId": 3}
== APP == Order received: {"orderId": 4}
== APP == Order received: {"orderId": 5}
== APP == Order received: {"orderId": 6}
== APP == Order received: {"orderId": 7}
== APP == Order received: {"orderId": 8}
== APP == Order received: {"orderId": 9}
== APP == Order received: {"orderId": 10}
```

{{% /codetab %}}

 <!-- JavaScript -->
{{% codetab %}}

### Step 1: Pre-requisites

对于此示例，您将需要：

- [Dapr CLI and initialized environment](https://docs.dapr.io/getting-started).
- [最新的Node.js已安装](https://nodejs.org/)。
<!-- IGNORE_LINKS -->
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
<!-- END_IGNORE -->

### 第2步：设置环境

克隆[快速入门存储库中提供的示例](https://github.com/dapr/quickstarts/tree/master/service_invocation)。

```bash
git clone https://github.com/dapr/quickstarts.git
```

### 第3步：运行 `order-processor` 服务

在终端窗口中，从快速入门克隆目录的根目录 导航到 `order-processor` 目录。

```bash
cd service_invocation/javascript/http/order-processor
```

安装依赖项：

```bash
npm install
```

Run the `order-processor` service alongside a Dapr sidecar.

```bash
dapr run --app-port 5001 --app-id order-processor --app-protocol http --dapr-http-port 3501 -- npm start
```

```javascript
app.post('/orders', (req, res) => {
    console.log("Order received:", req.body);
    res.sendStatus(200);
});
```

### 第4步：运行 `checkout` 服务

在新终端窗口中，从快速入门克隆目录的根目录导航到 `checkout` 目录。

```bash
cd service_invocation/javascript/http/checkout
```

安装依赖项：

```bash
npm install
```

与 Dapr sidecar 一起运行 `checkout` 服务。

```bash
dapr run --app-id checkout --app-protocol http --dapr-http-port 3500 -- npm start
```

在 `checkout` 服务中，您会注意到无需重写您的应用程序代码即可使用 Dapr 的服务调用。 您可以通过简单地添加 `dapr-app-id` 标头来启用服务调用，该标头指定目标服务的 ID。

```javascript
let axiosConfig = {
  headers: {
      "dapr-app-id": "order-processor"
  }
};
const res = await axios.post(`${DAPR_HOST}:${DAPR_HTTP_PORT}/orders`, order , axiosConfig);
console.log("Order passed: " + res.config.data);
```

### Step 5: Use with Multi-App Run

You can run the Dapr applications in this quickstart with the [Multi-App Run template]({{< ref multi-app-dapr-run >}}). Instead of running two separate `dapr run` commands for the `order-processor` and `checkout` applications, run the following command:

```sh
dapr run -f .
```

To stop all applications, run:

```sh
dapr stop -f .
```

### Step 6: View the Service Invocation outputs

Dapr invokes an application on any Dapr instance. In the code, the sidecar programming model encourages each application to talk to its own instance of Dapr. The Dapr instances then discover and communicate with one another.

`checkout` service output:

```
== APP == Order passed: {"orderId": 1}
== APP == Order passed: {"orderId": 2}
== APP == Order passed: {"orderId": 3}
== APP == Order passed: {"orderId": 4}
== APP == Order passed: {"orderId": 5}
== APP == Order passed: {"orderId": 6}
== APP == Order passed: {"orderId": 7}
== APP == Order passed: {"orderId": 8}
== APP == Order passed: {"orderId": 9}
== APP == Order passed: {"orderId": 10}
```

`order-processor` service output:

```
== APP == Order received: {"orderId": 1}
== APP == Order received: {"orderId": 2}
== APP == Order received: {"orderId": 3}
== APP == Order received: {"orderId": 4}
== APP == Order received: {"orderId": 5}
== APP == Order received: {"orderId": 6}
== APP == Order received: {"orderId": 7}
== APP == Order received: {"orderId": 8}
== APP == Order received: {"orderId": 9}
== APP == Order received: {"orderId": 10}
```

{{% /codetab %}}

 <!-- .NET -->
{{% codetab %}}

### Step 1: Pre-requisites

对于此示例，您将需要：

- [Dapr CLI and initialized environment](https://docs.dapr.io/getting-started).
- [.NET SDK or .NET 6 SDK installed](https://dotnet.microsoft.com/download).
<!-- IGNORE_LINKS -->
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
<!-- END_IGNORE -->

### 第2步：设置环境

克隆[快速入门存储库中提供的示例](https://github.com/dapr/quickstarts/tree/master/service_invocation)。

```bash
git clone https://github.com/dapr/quickstarts.git
```

### 第3步：运行 `order-processor` 服务

在终端窗口中，从快速入门克隆目录的根目录 导航到 `order-processor` 目录。

```bash
cd service_invocation/csharp/http/order-processor
```

安装依赖项：

```bash
dotnet restore
dotnet build
```

Run the `order-processor` service alongside a Dapr sidecar.

```bash
dapr run --app-port 7001 --app-id order-processor --app-protocol http --dapr-http-port 3501 -- dotnet run
```

Below is the working code block from the order processor's `Program.cs` file.

```csharp
app.MapPost("/orders", (Order order) =>
{
    Console.WriteLine("Order received : " + order);
    return order.ToString();
});
```

### 第4步：运行 `checkout` 服务

在新终端窗口中，从快速入门克隆目录的根目录导航到 `checkout` 目录。

```bash
cd service_invocation/csharp/http/checkout
```

安装依赖项：

```bash
dotnet restore
dotnet build
```

与 Dapr sidecar 一起运行 `checkout` 服务。

```bash
dapr run --app-id checkout --app-protocol http --dapr-http-port 3500 -- dotnet run
```

In the Program.cs file for the `checkout` service, you'll notice there's no need to rewrite your app code to use Dapr's service invocation. 您可以通过简单地添加 `dapr-app-id` 标头来启用服务调用，该标头指定目标服务的 ID。

```csharp
var client = new HttpClient();
client.DefaultRequestHeaders.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));

client.DefaultRequestHeaders.Add("dapr-app-id", "order-processor");

var response = await client.PostAsync($"{baseURL}/orders", content);
    Console.WriteLine("Order passed: " + order);
```

### Step 5: Use with Multi-App Run

You can run the Dapr applications in this quickstart with the [Multi-App Run template]({{< ref multi-app-dapr-run >}}). Instead of running two separate `dapr run` commands for the `order-processor` and `checkout` applications, run the following command:

```sh
dapr run -f .
```

To stop all applications, run:

```sh
dapr stop -f .
```

### Step 6: View the Service Invocation outputs

Dapr invokes an application on any Dapr instance. In the code, the sidecar programming model encourages each application to talk to its own instance of Dapr. The Dapr instances then discover and communicate with one another.

`checkout` service output:

```
== APP == Order passed: Order { OrderId: 1 }
== APP == Order passed: Order { OrderId: 2 }
== APP == Order passed: Order { OrderId: 3 }
== APP == Order passed: Order { OrderId: 4 }
== APP == Order passed: Order { OrderId: 5 }
== APP == Order passed: Order { OrderId: 6 }
== APP == Order passed: Order { OrderId: 7 }
== APP == Order passed: Order { OrderId: 8 }
== APP == Order passed: Order { OrderId: 9 }
== APP == Order passed: Order { OrderId: 10 }
```

`order-processor` service output:

```
== APP == Order received: Order { OrderId: 1 }
== APP == Order received: Order { OrderId: 2 }
== APP == Order received: Order { OrderId: 3 }
== APP == Order received: Order { OrderId: 4 }
== APP == Order received: Order { OrderId: 5 }
== APP == Order received: Order { OrderId: 6 }
== APP == Order received: Order { OrderId: 7 }
== APP == Order received: Order { OrderId: 8 }
== APP == Order received: Order { OrderId: 9 }
== APP == Order received: Order { OrderId: 10 }
```

{{% /codetab %}}

 <!-- Java -->
{{% codetab %}}

### Step 1: Pre-requisites

对于此示例，您将需要：

- [Dapr CLI and initialized environment](https://docs.dapr.io/getting-started).
- Java JDK 11 (or greater):
  - [Oracle JDK](https://www.oracle.com/java/technologies/downloads), or
  - OpenJDK
- [Apache Maven](https://maven.apache.org/install.html), version 3.x.
<!-- IGNORE_LINKS -->
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
<!-- END_IGNORE -->

### 第2步：设置环境

克隆[快速入门存储库中提供的示例](https://github.com/dapr/quickstarts/tree/master/service_invocation)。

```bash
git clone https://github.com/dapr/quickstarts.git
```

### 第3步：运行 `order-processor` 服务

在终端窗口中，从快速入门克隆目录的根目录 导航到 `order-processor` 目录。

```bash
cd service_invocation/java/http/order-processor
```

安装依赖项：

```bash
mvn clean install
```

Run the `order-processor` service alongside a Dapr sidecar.

```bash
dapr run --app-id order-processor --app-port 9001 --app-protocol http --dapr-http-port 3501 -- java -jar target/OrderProcessingService-0.0.1-SNAPSHOT.jar
```

```java
public String processOrders(@RequestBody Order body) {
        System.out.println("Order received: "+ body.getOrderId());
        return "CID" + body.getOrderId();
    }
```

### 第4步：运行 `checkout` 服务

在新终端窗口中，从快速入门克隆目录的根目录导航到 `checkout` 目录。

```bash
cd service_invocation/java/http/checkout
```

安装依赖项：

```bash
mvn clean install
```

与 Dapr sidecar 一起运行 `checkout` 服务。

```bash
dapr run --app-id checkout --app-protocol http --dapr-http-port 3500 -- java -jar target/CheckoutService-0.0.1-SNAPSHOT.jar
```

在 `checkout` 服务中，您会注意到无需重写您的应用程序代码即可使用 Dapr 的服务调用。 您可以通过简单地添加 `dapr-app-id` 标头来启用服务调用，该标头指定目标服务的 ID。

```java
.header("Content-Type", "application/json")
.header("dapr-app-id", "order-processor")

HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println("Order passed: "+ orderId)
```

### Step 5: Use with Multi-App Run

You can run the Dapr applications in this quickstart with the [Multi-App Run template]({{< ref multi-app-dapr-run >}}). Instead of running two separate `dapr run` commands for the `order-processor` and `checkout` applications, run the following command:

```sh
dapr run -f .
```

To stop all applications, run:

```sh
dapr stop -f .
```

### Step 6: View the Service Invocation outputs

Dapr invokes an application on any Dapr instance. In the code, the sidecar programming model encourages each application to talk to its own instance of Dapr. The Dapr instances then discover and communicate with one another.

`checkout` service output:

```
== APP == Order passed: 1
== APP == Order passed: 2
== APP == Order passed: 3
== APP == Order passed: 4
== APP == Order passed: 5
== APP == Order passed: 6
== APP == Order passed: 7
== APP == Order passed: 8
== APP == Order passed: 9
== APP == Order passed: 10
```

`order-processor` service output:

```
== APP == Order received: 1
== APP == Order received: 2
== APP == Order received: 3
== APP == Order received: 4
== APP == Order received: 5
== APP == Order received: 6
== APP == Order received: 7
== APP == Order received: 8
== APP == Order received: 9
== APP == Order received: 10
```

{{% /codetab %}}

 <!-- Go -->
{{% codetab %}}

### Step 1: Pre-requisites

对于此示例，您将需要：

- [Dapr CLI and initialized environment](https://docs.dapr.io/getting-started).
- [Latest version of Go](https://go.dev/dl/).
<!-- IGNORE_LINKS -->
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
<!-- END_IGNORE -->

### 第2步：设置环境

克隆[快速入门存储库中提供的示例](https://github.com/dapr/quickstarts/tree/master/service_invocation)。


```bash
git clone https://github.com/dapr/quickstarts.git
```

### 第3步：运行 `order-processor` 服务

在终端窗口中，从快速入门克隆目录的根目录 导航到 `order-processor` 目录。

```bash
cd service_invocation/go/http/order-processor
```

安装依赖项：

```bash
go build .
```

Run the `order-processor` service alongside a Dapr sidecar.

```bash
dapr run --app-port 6001 --app-id order-processor --app-protocol http --dapr-http-port 3501 -- go run .
```

Each order is received via an HTTP POST request and processed by the `getOrder` function.

```go
func getOrder(w http.ResponseWriter, r *http.Request) {
    data, err := ioutil.ReadAll(r.Body)
    if err != nil {
        log.Fatal(err)
    }
    log.Printf("Order received : %s", string(data))
}
```

### 第4步：运行 `checkout` 服务

在新终端窗口中，从快速入门克隆目录的根目录导航到 `checkout` 目录。

```bash
cd service_invocation/go/http/checkout
```

安装依赖项：

```bash
go build .
```

与 Dapr sidecar 一起运行 `checkout` 服务。

```bash
dapr run --app-id checkout --app-protocol http --dapr-http-port 3500 -- go run .
```

在 `checkout` 服务中，您会注意到无需重写您的应用程序代码即可使用 Dapr 的服务调用。 您可以通过简单地添加 `dapr-app-id` 标头来启用服务调用，该标头指定目标服务的 ID。

```go
req.Header.Add("dapr-app-id", "order-processor")

response, err := client.Do(req)
```

### Step 5: Use with Multi-App Run

You can run the Dapr applications in this quickstart with the [Multi-App Run template]({{< ref multi-app-dapr-run >}}). Instead of running two separate `dapr run` commands for the `order-processor` and `checkout` applications, run the following command:

```sh
dapr run -f .
```

To stop all applications, run:

```sh
dapr stop -f .
```

### Step 6: View the Service Invocation outputs

Dapr invokes an application on any Dapr instance. In the code, the sidecar programming model encourages each application to talk to its own instance of Dapr. The Dapr instances then discover and communicate with one another.

`checkout` service output:

```
== APP == Order passed:  {"orderId":1}
== APP == Order passed:  {"orderId":2}
== APP == Order passed:  {"orderId":3}
== APP == Order passed:  {"orderId":4}
== APP == Order passed:  {"orderId":5}
== APP == Order passed:  {"orderId":6}
== APP == Order passed:  {"orderId":7}
== APP == Order passed:  {"orderId":8}
== APP == Order passed:  {"orderId":9}
== APP == Order passed:  {"orderId":10}
```

`order-processor` service output:

```
== APP == Order received :  {"orderId":1}
== APP == Order received :  {"orderId":2}
== APP == Order received :  {"orderId":3}
== APP == Order received :  {"orderId":4}
== APP == Order received :  {"orderId":5}
== APP == Order received :  {"orderId":6}
== APP == Order received :  {"orderId":7}
== APP == Order received :  {"orderId":8}
== APP == Order received :  {"orderId":9}
== APP == Order received :  {"orderId":10}
```


{{% /codetab %}}

{{% /tabs %}}

## Tell us what you think!
We're continuously working to improve our Quickstart examples and value your feedback. Did you find this Quickstart helpful? Do you have suggestions for improvement?

Join the discussion in our [discord channel](https://discord.com/channels/778680217417809931/953427615916638238).

## 下一步

- Learn more about [Service Invocation as a Dapr building block]({{< ref service-invocation-overview.md >}})
- 了解更多关于如何调用 Dapr 的服务调用：
    - [HTTP]({{< ref howto-invoke-discover-services.md >}}), or
    - [gRPC]({{< ref howto-invoke-services-grpc.md >}})

{{< button text="Explore Dapr tutorials  >>" page="getting-started/tutorials/_index.md" >}}