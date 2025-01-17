---
type: docs
title: "Azure Cosmos DB (SQL API)"
linkTitle: "Azure Cosmos DB (SQL API)"
description: Detailed information on the Azure Cosmos DB (SQL API) state store component
aliases:
  - "/zh-hans/operations/components/setup-state-store/supported-state-stores/setup-azure-cosmosdb/"
---

## Component format

要设置 Azure CosmosDb 状态存储，请创建一个类型为 `state.azure.cosmosdb` 的组件。 See [this guide]({{< ref "howto-get-save-state.md#step-1-setup-a-state-store" >}}) on how to create and apply a state store configuration.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <NAME>
spec:
  type: state.azure.cosmosdb
  version: v1
  metadata:
  - name: url
    value: <REPLACE-WITH-URL>
  - name: masterKey
    value: <REPLACE-WITH-MASTER-KEY>
  - name: database
    value: <REPLACE-WITH-DATABASE>
  - name: collection
    value: <REPLACE-WITH-COLLECTION>
```

{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件，参考 [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

如果您想要使用 CosmosDb 作为 Actor 存储，请在 yaml 上附上以下内容。

```yaml
  - name: actorStateStore
    value: "true"
```

## 元数据字段规范

| Field           | 必填 | 详情                                                                                                      | 示例                                           |
| --------------- |:--:| ------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| url             | 是  | The Cosmos DB url                                                                                       | `"https://******.documents.azure.com:443/"`. |
| masterKey       | Y* | The key to authenticate to the Cosmos DB account. Only required when not using Azure AD authentication. | `"key"`                                      |
| database        | 是  | 数据库名称                                                                                                   | `"db"`                                       |
| collection      | 是  | 要使用的集合（容器）名称                                                                                            | `"collection"`                               |
| actorStateStore | 否  | 是否将此状态存储给 Actor 使用。 默认值为 `"false"`                                                                      | `"true"`, `"false"`                          |

### Azure Active Directory (Azure AD) 认证

Azure Cosmos DB状态存储组件支持使用所有Azure Active Directory机制进行认证。 关于更多信息和相关组件的元数据字段请根据选择的Azure AD认证机制，参考[Azure认证文档]({{< ref authenticating-azure.md >}})。

您可以在</a>下面的
部分中阅读有关使用 Azure AD 身份验证设置 Cosmos DB 的其他信息。</p> 



## 设置Azure Cosmos DB

[请遵循 Azure 文档中关于如何创建 Azure CosmosDB 账户的说明](https://docs.microsoft.com/azure/cosmos-db/how-to-manage-database-account)。  在为 Dapr 所使用之前，必须先在 CosmosDB 中创建数据库和集合。

**重要提示：集合的分区键必须命名为 `/partitionKey` （注意：这是区分大小写的）。**

为了配置 CosmosDB 作为状态存储，你需要以下属性：

- **URL**: the CosmosDB url. 例如： `https://******.documents.azure.com:443/`
- **Master Key**: The key to authenticate to the Cosmos DB account. Skip this if using Azure AD authentication.
- **Database**: The name of the database
- **Collection**: 集合（或者容器）的名称



## 生产使用的最佳实践

Azure Cosmos DB shares a strict metadata request rate limit across all databases in a single Azure Cosmos DB account. New connections to Azure Cosmos DB assume a large percentage of the allowable request rate limit. (See the [Cosmos DB documentation](https://docs.microsoft.com/azure/cosmos-db/sql/troubleshoot-request-rate-too-large#recommended-solution-3))

Therefore several strategies must be applied to avoid simultaneous new connections to Azure Cosmos DB:

- 确保应用程序的 sidecar 仅在需要时加载 Azure Cosmos DB 组件，以避免不必要的数据库连接。 This can be done by [scoping your components to specific applications]({{< ref component-scopes.md >}}#application-access-to-components-with-scopes).
- 选择按顺序部署或启动应用程序的部署策略，以最大程度地减少 Azure Cosmos DB 账户新连接造成的影响。
- 避免对不相关的数据库或系统（甚至在 Dapr 外部）重用同一 Azure Cosmos DB 账户。 不同的 Azure Cosmos DB 账户具有不同的速率限制。
- 增加 `initTimeout` 值，以允许组件在 sidecar 初始化期间重试连接到 Azure Cosmos DB，最长5分钟。 默认值是 `5s` ，应该增加。 使用 Kubernetes 时，增加此值可能还需要更新您的 [Readiness 和 Liveness 探针](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)。



```yaml
spec:
  type: state.azure.cosmosdb
  version: v1
  initTimeout: 5m
  metadata:
```




## Data format

要使用CosmosDB状态存储，你的数据必须以JSON序列化的方式发送到Dapr。 Having it just JSON *serializable* will not work.

如果您使用 Dapr SDK（例如 [.NET SDK](https://github.com/dapr/dotnet-sdk)），SDK 会自动将您的数据序列化为 JSON。

如果您想直接调用 Dapr 的 HTTP 端点，请查看下面 [Partition keys](#partition-keys) 部分中的示例（使用 curl）。



## Partition keys

对于**non-actor**状态操作，Azure Cosmos DB状态存储将使用向Dapr API发出的请求中提供的`key`属性来确定Cosmos DB分区键。 This can be overridden by specifying a metadata field in the request with a key of `partitionKey` and a value of the desired partition.

以下操作将使用`nihilus`作为发送到CosmosDB的分区键值：



```shell
curl -X POST http://localhost:3500/v1.0/state/<store_name> \
  -H "Content-Type: application/json"
  -d '[
        {
          "key": "nihilus",
          "value": "darth"
        }
      ]'
```


对于**non-actor**状态操作，如果你想控制CosmosDB分区，你可以在元数据中指定它。  Reusing the example above, here's how to put it under the `mypartition` partition



```shell
curl -X POST http://localhost:3500/v1.0/state/<store_name> \
  -H "Content-Type: application/json"
  -d '[
        {
          "key": "nihilus",
          "value": "darth",
          "metadata": {
            "partitionKey": "mypartition"
          }
        }
      ]'
```


For **actor** state operations, the partition key is generated by Dapr using the `appId`, the actor type, and the actor id, such that data for the same actor always ends up under the same partition (you do not need to specify it). 这是因为actor状态操作必须使用事务，而在CosmosDB中，事务中的项必须在同一个分区上。



## Setting up Cosmos DB for authenticating with Azure AD

当使用Dapr Cosmos DB状态存储组件并使用Azure AD进行身份认证时，你需要执行一些额外的步骤去设置你的环境。

Prerequisites:

- You need a Service Principal created as per the instructions in the [authenticating to Azure]({{< ref authenticating-azure.md >}}) page. You need the ID of the Service Principal for the commands below (note that this is different from the client ID of your application, or the value you use for `azureClientId` in the metadata).
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [jq](https://stedolan.github.io/jq/download/)
- The scripts below are optimized for a bash or zsh shell



### 授予 Azure AD 应用程序对 Cosmos DB 的访问权限



> You can find more information on the [official documentation](https://docs.microsoft.com/azure/cosmos-db/how-to-setup-rbac), including instructions to assign more granular permissions.

In order to grant your application permissions to access data stored in Cosmos DB, you need to assign it a custom role for the Cosmos DB data plane. In this example you're going to use a built-in role, "Cosmos DB Built-in Data Contributor", which grants your application full read-write access to the data; you can optionally create custom, fine-tuned roles following the instructions in the official docs.



```sh
# Name of the Resource Group that contains your Cosmos DB
RESOURCE_GROUP="..."
# Name of your Cosmos DB account
ACCOUNT_NAME="..."
# ID of your Service Principal object
PRINCIPAL_ID="..."
# ID of the "Cosmos DB Built-in Data Contributor" role
# You can also use the ID of a custom role
ROLE_ID="00000000-0000-0000-0000-000000000002"

az cosmosdb sql role assignment create \
  --account-name "$ACCOUNT_NAME" \
  --resource-group "$RESOURCE_GROUP" \
  --scope "/" \
  --principal-id "$PRINCIPAL_ID" \
  --role-definition-id "$ROLE_ID"
```




## 相关链接

- [Basic schema for a Dapr component]({{< ref component-schema >}})
- 阅读 [本指南]({{< ref "howto-get-save-state.md#step-2-save-and-retrieve-a-single-state" >}}) 以获取配置状态存储组件的说明
- [状态管理构建块]({{< ref state-management >}})
