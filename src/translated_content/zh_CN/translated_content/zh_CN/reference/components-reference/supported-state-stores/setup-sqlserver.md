---
type: docs
title: "SQL Server"
linkTitle: "SQL Server"
description: SQL Server 状态存储组件的详细信息
aliases:
  - "/zh-hans/operations/components/setup-state-store/supported-state-stores/setup-sqlserver/"
---

## Component format

To setup SQL Server state store create a component of type `state.sqlserver`. See [this guide]({{< ref "howto-get-save-state.md#step-1-setup-a-state-store" >}}) on how to create and apply a state store configuration.


```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <NAME>
spec:
  type: state.sqlserver
  version: v1
  metadata:
  - name: connectionString
    value: <REPLACE-WITH-CONNECTION-STRING> # Required.
  - name: tableName
    value: <REPLACE-WITH-TABLE-NAME>  # Optional. defaults to "state"
  - name: keyType
    value: <REPLACE-WITH-KEY-TYPE>  # Optional. defaults to "string"
  - name: keyLength
    value: <KEY-LENGTH> # Optional. defaults to 200. You be used with "string" keyType
  - name: schema
    value: <SCHEMA> # Optional. defaults to "dbo"
  - name: indexedProperties
    value: <INDEXED-PROPERTIES> # Optional. List of IndexedProperties.

```

{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件，参考 [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

如果您想要使用 SQL Server 作为 [actor 状态存储]({{< ref "state_api.md#configuring-state-store-for-actors" >}}) ，请在 yaml 上附上以下内容。

```yaml
  - name: actorStateStore
    value: "true"
```

## 元数据字段规范

| Field             | 必填 | 详情                                                                                                                                                                                    | 示例                                                                                                                                            |
| ----------------- |:--:| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| connectionString  | 是  | The connection string used to connect. If the connection string contains the database it must already exist. If the database is omitted a default database named `"Dapr"` is created. | `"Server=myServerName\myInstanceName;Database=myDataBase;User Id=myUsername;Password=myPassword;"`                                           |
| tableName         | 否  | 要使用的表名称。 带下划线的字母数字。 默认值为 `"state"`                                                                                                                                                    | `"table_name"`                                                                                                                                |
| keyType           | 否  | 键使用的数据类型。 默认为 `"string"`                                                                                                                                                              | `"string"`                                                                                                                                    |
| keyLength         | 否  | 键的最大长度。 与 `"string"` keyType 一起使用。 Defaults to `"200"`                                                                                                                                | `"200"`                                                                                                                                       |
| schema            | 否  | 要使用的schema名称。 默认为 `"dbo"`                                                                                                                                                             | `"dapr"`,`"dbo"`                                                                                                                              |
| indexedProperties | 否  | 索引属性列表。                                                                                                                                                                               | `'[{"column": "transactionid", "property": "id", "type": "int"}, {"column": "customerid", "property": "customer", "type": "nvarchar(100)"}]'` |
| actorStateStore   | 否  | 指示 Dapr 是否应该将为 actor 状态存储配置该组件 ([更多信息]({{< ref "state_api.md#configuring-state-store-for-actors" >}}))。                                                                               | `"true"`                                                                                                                                      |


## 创建 Azure SQL 实例

按照 Azure 文档中有关如何创建 SQL 数据库的说明[进行操作](https://docs.microsoft.com/azure/azure-sql/database/single-database-create-quickstart?view=azuresql&tabs=azure-portal) 。  必须在 Dapr 使用数据库之前创建数据库。

**注意：SQL Server 状态存储还支持在 VM 和 Docker 中运行 SQL Server。**

为了配置 SQL Server 作为状态存储，您需要如下属性：

- **Connection String**: The SQL Server connection string. For example: server=localhost;user id=sa;password=your-password;port=1433;database=mydatabase;
- **Schema**: The database schema to use (default=dbo). Will be created if does not exist
- **Table Name**: The database table name. Will be created if does not exist
- **Indexed Properties**: Optional properties from json data which will be indexed and persisted as individual column

### 创建专用用户

当使用专用用户 (不是 `sa`) 进行连接， 用户需要这些授权 - 即使该用户是所需数据的模式的所有者：

- `CREATE TABLE`
- `CREATE TYPE`

## 相关链接
- [Basic schema for a Dapr component]({{< ref component-schema >}})
- 阅读 [本指南]({{< ref "howto-get-save-state.md#step-2-save-and-retrieve-a-single-state" >}}) 以获取配置状态存储组件的说明
- [状态管理构建块]({{< ref state-management >}})