---
type: docs
title: "使用 Kubernetes 作业运行 Dapr"
linkTitle: "Kubernetes Jobs"
weight: 60000
description: "在 Kubernetes 作业上下文中使用 Dapr API"
---

# Kubernetes Job

The Dapr sidecar is designed to be a long running process, in the context of a [Kubernetes Job](https://kubernetes.io/docs/concepts/workloads/controllers/job/) this behaviour can block your job completion. To address this issue the Dapr sidecar has an endpoint to `Shutdown` the sidecar.

在运行基本 [Kubernetes 作业](https://kubernetes.io/docs/concepts/workloads/controllers/job/) 时，您需要调用 `/shutdown` 端点，以便 sidecar 正常停止，并且作业将被视为 `Completed`。

When a job is finished without calling `Shutdown`, your job will be in a `NotReady` state with only the `daprd` container running endlessly.

Stopping the dapr sidecar will cause its readiness and liveness probes to fail in your container because the dapr sidecar was shutdown. To prevent Kubernetes from trying to restart your job, set your job's `restartPolicy` to `Never`.

Be sure to use the *POST* HTTP verb when calling the shutdown HTTP API.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-with-shutdown
spec:
  template:
    metadata:
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "with-shutdown"
    spec:
      containers:
      - name: job
        image: alpine
        command: ["/bin/sh", "-c", "apk --no-cache add curl && sleep 20 && curl -X POST localhost:3500/v1.0/shutdown"]
      restartPolicy: Never
```

You can also call the `Shutdown` from any of the Dapr SDKs

```go
package main

import (
    "context"
    "log"
    "os"

    dapr "github.com/dapr/go-sdk/client"
)

func main() {
  client, err := dapr.NewClient()
  if err != nil {
    log.Panic(err)
  }
  defer client.Close()
  defer client.Shutdown()
  // Job
}
```
