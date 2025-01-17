---
type: docs
title: "使用PHP的状态管理"
linkTitle: "状态管理"
weight: 1000
description: 使用方式
no_list: true
---

Dapr offers a great modular approach to using state in your application. The best way to learn the basics is to visit [the howto]({{< ref howto-get-save-state.md >}}).

## Metadata

许多状态组件允许您将元数据传递给组件，以控制组件的特定行为。 PHP SDK 允许您通过以下方式传递该元数据：

```php
<?php
// using the state manager
$app->run(
    fn(\Dapr\State\StateManager $stateManager) => 
        $stateManager->save_state('statestore', new \Dapr\State\StateItem('key', 'value', metadata: ['port' => '112'])));

// using the DaprClient
$app->run(fn(\Dapr\Client\DaprClient $daprClient) => $daprClient->saveState(storeName: 'statestore', key: 'key', value: 'value', metadata: ['port' => '112']))
```

这是如何将端口元数据传递给 [cassandra]({{< ref setup-cassandra.md >}}) 的示例。

每个状态操作都允许传递元数据。

## 一致性/并发性

在 PHP SDK 中，Dapr 中有四种不同类型的一致性和并发性：

```php
<?php
[
    \Dapr\consistency\StrongLastWrite::class, 
    \Dapr\consistency\StrongFirstWrite::class,
    \Dapr\consistency\EventualLastWrite::class,
    \Dapr\consistency\EventualFirstWrite::class,
] 
```

将其中之一传递给 `StateManager` 方法或使用 `StateStore()` 属性允许您定义状态存储应该如何处理冲突。

## 并行

进行批量读取或开始事务时，可以指定并行的数量。 如果 Dapr 不得不一次只读取一个键，它将"最多"能同时从底层存储中读取这么多键。 这有助于控制状态存储上的负载，但会牺牲性能。 该属性的默认值是 `10`。

## 前缀

硬编码的键名很有用，但是为什么不让状态对象可复用性更高呢？ When committing a transaction or saving an object to state, you can pass a prefix that is applied to every key in the object.

{{< tabs "Transaction prefix" "StateManager prefix" >}}

{{% codetab %}}

```php
<?php
class TransactionObject extends \Dapr\State\TransactionalState {
    public string $key;
}

$app->run(function (TransactionObject $object ) {
    $object->begin(prefix: 'my-prefix-');
    $object->key = 'value';
    // commit to key `my-prefix-key`
    $object->commit();
});
```

{{% /codetab %}}
{{% codetab %}}

```php
<?php
class StateObject {
    public string $key;
}

$app->run(function(\Dapr\State\StateManager $stateManager) {
    $stateManager->load_object($obj = new StateObject(), prefix: 'my-prefix-');
    // original value is from `my-prefix-key`
    $obj->key = 'value';
    // save to `my-prefix-key`
    $stateManager->save_object($obj, prefix: 'my-prefix-');
});
```

{{% /codetab %}}

{{< /tabs >}}
