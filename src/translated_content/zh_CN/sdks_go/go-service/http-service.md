---
type: docs
title: "Dapr HTTP 服务 SDK for Go 入门"
linkTitle: "HTTP 服务"
weight: 10000
description: 如何使 Dapr HTTP 服务 SDK for Go 启动和运行
no_list: true
---

### Prerequisite
Start by importing Dapr Go service/http package:

```go
daprd "github.com/dapr/go-sdk/service/http"
```

### 创建和启动服务
要创建 HTTP Dapr 服务，首先创建一个具有特定地址的 Dapr 回调实例：

```go
s := daprd.NewService(":8080")
```

如果想合并一个已存在的服务，可以用该服务的的地址和http.ServeMux:

```go
mux := http.NewServeMux()
mux.HandleFunc("/", myOtherHandler)
s := daprd.NewServiceWithMux(":8080", mux)
```

一旦你创建了一个服务实例，你就可以给该服务 "附加 "任何数量的事件、绑定和服务调用逻辑处理程序。 只要逻辑定义好，即可启动服务：

```go
if err := s.Start(); err != nil && err != http.ErrServerClosed {
    log.Fatalf("error: %v", err)
}
```

### 事件处理
要处理来自特定主题的事件，您需要在启动服务之前至少添加一个主题事件handler：

```go
sub := &common.Subscription{
    PubsubName: "messages",
    Topic:      "topic1",
    Route:      "/events",
}
err := s.AddTopicEventHandler(sub, eventHandler)
if err != nil {
    log.Fatalf("error adding topic subscription: %v", err)
}
```

handler 本身可以是具有预期签名的任何方法：

```go
func eventHandler(ctx context.Context, e *common.TopicEvent) (retry bool, err error) {
    log.Printf("event - PubsubName:%s, Topic:%s, ID:%s, Data: %v", e.PubsubName, e.Topic, e.ID, e.Data)
    // do something with the event
    return true, nil
}
```

Optionally, you can use [routing rules](https://docs.dapr.io/developing-applications/building-blocks/pubsub/howto-route-messages/) to send messages to different handlers based on the contents of the CloudEvent.

```go
sub := &common.Subscription{
    PubsubName: "messages",
    Topic:      "topic1",
    Route:      "/important",
    Match:      `event.type == "important"`,
    Priority:   1,
}
err := s.AddTopicEventHandler(sub, importantHandler)
if err != nil {
    log.Fatalf("error adding topic subscription: %v", err)
}
```

### 服务调用处理
To handle service invocations you will need to add at least one service invocation handler before starting the service:

```go
if err := s.AddServiceInvocationHandler("/echo", echoHandler); err != nil {
    log.Fatalf("error adding invocation handler: %v", err)
}
```

handler 本身可以是具有预期签名的任何方法：


```go
func echoHandler(ctx context.Context, in *common.InvocationEvent) (out *common.Content, err error) {
    log.Printf("echo - ContentType:%s, Verb:%s, QueryString:%s, %+v", in.ContentType, in.Verb, in.QueryString, string(in.Data))
    // do something with the invocation here 
    out = &common.Content{
        Data:        in.Data,
        ContentType: in.ContentType,
        DataTypeURL: in.DataTypeURL,
    }
    return
}
```

### 绑定调用处理

```go
if err := s.AddBindingInvocationHandler("/run", runHandler); err != nil {
    log.Fatalf("error adding binding handler: %v", err)
}
```

handler 本身可以是具有预期签名的任何方法：

```go
func runHandler(ctx context.Context, in *common.BindingEvent) (out []byte, err error) {
    log.Printf("binding - Data:%v, Meta:%v", in.Data, in.Metadata)
    // do something with the invocation here 
    return nil, nil
}
```
## 相关链接
- [Go SDK 示例](https://github.com/dapr/go-sdk/tree/main/examples)
