---
type: docs
title: "Postgres"
linkTitle: "Postgres"
description: Detailed information on the Postgres configuration store component
aliases:
  - "/zh-hans/operations/components/setup-configuration-store/supported-configuration-stores/setup-postgres/"
---

## Component format

To set up an Postgres configuration store, create a component of type `configuration.postgres`

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <NAME>
spec:
  type: configuration.postgres
  version: v1
  metadata:
  - name: connectionString
    value: "host=localhost user=postgres password=example port=5432 connect_timeout=10 database=config"
  - name: table # name of the table which holds configuration information
    value: "[your_configuration_table_name]" 
  - name: connMaxIdleTime # max timeout for connection
    value : "15s"

```

{{% alert title="Warning" color="warning" %}}
以上示例将密钥明文存储， 更推荐的方式是使用 Secret 组件，参考 [这里]({{< ref component-secrets.md >}})。
{{% /alert %}}

## 元数据字段规范

| Field            | 必填 | 详情                                                                 | 示例                                                                                                                  |
| ---------------- |:--:| ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| connectionString | 是  | The connection string for PostgreSQL. Default pool_max_conns = 5 | `"host=localhost user=postgres password=example port=5432 connect_timeout=10 database=dapr_test pool_max_conns=10"` |
| table            | 是  | table name for configuration information.                          | `configTable`                                                                                                       |

## Set up Postgres as Configuration Store

1. Start Postgres Database
1. Connect to the Postgres database and setup a configuration table with following schema -

| Field    | Datatype | Nullable | 详情                                           |
| -------- |:--------:| -------- | -------------------------------------------- |
| KEY      | VARCHAR  | 否        | Holds `"Key"` of the configuration attribute |
| VALUE    | VARCHAR  | 否        | Holds Value of the configuration attribute   |
| VERSION  | VARCHAR  | 否        | Holds version of the configuration attribute |
| METADATA |   JSON   | 是        | Holds Metadata as JSON                       |

```console
CREATE TABLE IF NOT EXISTS table_name (
        KEY VARCHAR NOT NULL,
        VALUE VARCHAR NOT NULL,
        VERSION VARCHAR NOT NULL,
        METADATA JSON );
```
3. Create a TRIGGER on configuration table. An example function to create a TRIGGER is as follows -
```console
CREATE OR REPLACE FUNCTION configuration_event() RETURNS TRIGGER AS $$
    DECLARE 
        data json;
        notification json;

    BEGIN

        IF (TG_OP = 'DELETE') THEN
            data = row_to_json(OLD);
        ELSE
            data = row_to_json(NEW);
        END IF;

        notification = json_build_object(
                          'table',TG_TABLE_NAME,
                          'action', TG_OP,
                          'data', data);

        PERFORM pg_notify('config',notification::text);
        RETURN NULL; 
    END;  
$$ LANGUAGE plpgsql;
```
4. Create the trigger with data encapsulated in the field labelled as `data`
```ps
notification = json_build_object(
                          'table',TG_TABLE_NAME,
                          'action', TG_OP,
                          'data', data);
```
5. The channel mentioned as attribute to `pg_notify` should be used when subscribing for configuration notifications
6. Since this is a generic created trigger, map this trigger to `configuration table`
```console
CREATE TRIGGER config
AFTER INSERT OR UPDATE OR DELETE ON configTable
    FOR EACH ROW EXECUTE PROCEDURE notify_event();
```
7. In the subscribe request add an additional metadata field with key as `pgNotifyChannel` and value should be set to same `channel name` mentioned in `pg_notify`. From the above example, it should be set to `config`

{{% alert title="Note" color="primary" %}}
When calling `subscribe` API, `metadata.pgNotifyChannel` should be used to specify the name of the channel to listen for notifications from Postgres configuration store.

Any number of keys can be added to a subscription request. Each subscription uses an exclusive database connection. It is strongly recommended to subscribe to multiple keys within a single subscription. This helps optimize the number of connections to the database.

Example of subscribe HTTP API -
```ps
curl --location --request GET 'http://<host>:<dapr-http-port>/configuration/postgres/subscribe?key=<keyname1>&key=<keyname2>&metadata.pgNotifyChannel=<channel name>'
```
{{% /alert %}}

## 相关链接
- [Basic schema for a Dapr component]({{< ref component-schema >}})
- [配置构建基块]({{< ref configuration-api-overview >}})
