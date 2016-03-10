Ignite supports 2 modes for cache operation, *transactional* and *atomic*. In `transactional` mode you are able to group multiple cache operations in a transaction, while `atomic` mode supports multiple atomic operations, one at a time. `Atomic` mode is more light-weight and generally has better performance over `transactional` caches.

However, regardless of which mode you use, as long as your cluster is alive, the data between different cluster nodes must remain consistent. This means that whichever node is being used to retrieve data, it will never get data that has been partially committed or that is inconsistent with other data.
[block:api-header]
{
  "type": "basic",
  "title": "IgniteTransactions"
}
[/block]
`IgniteTransactions` interface contains functionality for starting and completing transactions, as well as subscribing listeners or getting metrics.
[block:callout]
{
  "type": "info",
  "title": "Cross-Cache Transactions",
  "body": "You can combine multiple operations from different caches into one transaction. Note that this allows to update caches of different types, like `REPLICATED` and `PARTITIONED` caches, in one transaction."
}
[/block]
You can obtain an instance of `IgniteTransactions` as follows:
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nIgniteTransactions transactions = ignite.transactions();",
      "language": "java"
    }
  ]
}
[/block]
Here is an example of how transactions can be performed in Ignite:
[block:code]
{
  "codes": [
    {
      "code": "try (Transaction tx = transactions.txStart()) {\n    Integer hello = cache.get(\"Hello\");\n  \n    if (hello == 1)\n        cache.put(\"Hello\", 11);\n  \n    cache.put(\"World\", 22);\n  \n    tx.commit();\n}",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Two-Phase-Commit (2PC)"
}
[/block]
Ignite utilizes 2PC protocol for its transactions with many one-phase-commit optimizations whenever applicable. Whenever data is updated within a transaction, Ignite will keep transactional state in a local transaction map until `commit()` is called, at which point, if needed, the data is transferred to participating remote nodes.

For more information on how Ignite 2PC works, you can check out these blogs:
  * [Two-Phase-Commit for Distributed In-Memory Caches](http://gridgain.blogspot.com/2014/09/two-phase-commit-for-distributed-in.html)
  *  [Two-Phase-Commit for In-Memory Caches - Part II](http://gridgain.blogspot.com/2014/09/two-phase-commit-for-in-memory-caches.html) 
  * [One-Phase-Commit - Fast Transactions For In-Memory Caches](http://gridgain.blogspot.com/2014/09/one-phase-commit-fast-transactions-for.html) 
[block:callout]
{
  "type": "success",
  "body": "Ignite provides fully ACID (**A**tomicity, **C**onsistency, **I**solation, **D**urability) compliant transactions that ensure guaranteed consistency.",
  "title": "ACID Compliance"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Optimistic and Pessimistic"
}
[/block]
Whenever `TRANSACTIONAL` atomicity mode is configured, Ignite supports `OPTIMISTIC` and `PESSIMISTIC` concurrency modes for transactions. Locking prevents concurrent access to an object. For example, when you attempt to update a ToDo list item with pessimistic locking, the server places a lock on the object until you either commit or rollback the transaction so that no other transaction or operation is allowed to update the same entry. `OPTIMISTIC` locking is an application-side check on whether the timestamp/version of a record has changed between fetching and attempting to update it. This is locking configuration is regardless of transaction isolation level.
[block:callout]
{
  "type": "info",
  "body": "The main difference is that in `PESSIMISTIC` mode locks are acquired at the time of access, while in `OPTIMISTIC` mode locks are acquired during the `commit` phase.",
  "title": "Optimistic and Pessimistic Locking"
}
[/block]
Ignite also supports the following isolation levels:
  * `READ_COMMITED` - data is always fetched from the primary node, even if it already has been accessed within the transaction. In this isolation you can have so-called Non-Repeatable Reads because someone else can change the data when you are reading the data twice in your transaction. In `PESSIMISTIC` mode it means that the lock is only held at the time of access and released soon after. This cannot guarantee that the data is same in every consecutive read even within the same transaction. As for `OPTIMISTIC` mode use this isolation level only if you're sure that there won't be two concurrent transactions that work with intersecting sets of keys, otherwise the result of a transaction is undefined. 

  * `REPEATABLE_READ` - data is fetched form the primary node only once on first access and stored in the local transactional map. All consecutive access to the same data is local. In `PESSIMISTIC` mode the Server holds the lock until you end your transaction with a COMMIT or ROLLBACK. This means nobody else can make changes to your read data, and you are getting Repeatable Reads for your transaction. As for `OPTIMISTIC` mode use this isolation level only if you're sure that there won't be two concurrent transactions that work with intersecting sets of keys, otherwise the result of a transaction is undefined.

  * `SERIALIZABLE` - in `PESSIMISTIC` mode this isolation level works the same way as with `REPEATABLE_READ`. This is a primary isolation level that has to be used with `OPTIMISTIC` mode. Using this isolation level and `OPTIMISTIC` mode Ignite enables an algorithm that detects possible concurrent updates at `commit` phase of a transaction and may throw `TransactionOptimisticException` in case if a concurrent update happened from the other transaction.
[block:api-header]
{
  "type": "basic",
  "title": "Deadlock-free Transactions"
}
[/block]
`OPTIMISTIC` `SERIALIZABLE` transactions provide you with an ability to work with deadlock-free transactions. This is feasible since Ignite will fail a transaction at the commit stage if the Ignite engine detects that at least one of the entries used as  part of the initiated transaction has been modified. This is achieved by internally checking the version of an entry used in a transaction to the one actually in the grid at the time of commit. In short this means that if Ignite detects that there is a conflict at the commit stage of a transaction we fail such a transaction throwing `TransactionOptimisticException` & rolling back any changes made. By handling this exception you may then implement retry mechanisms or any other logic required
[block:code]
{
  "codes": [
    {
      "code": "IgniteTransactions txs = ignite.transactions();\n\n// Start transaction in optimistic mode with serializable isolation level.\nwhile (true) {\n    try (Transaction tx =  \n         ignite.transactions().txStart(TransactionConcurrency.OPTIMISTIC,\n                                       TransactionIsolation.SERIALIZABLE)) {\n\t \t\t\t// Modify cache entires as part of this transcation.\n  \t\t\t....\n        \n  \t\t\t// commit transaction.  \n  \t\t\ttx.commit();\n\n      \t// Transaction succeeded. Leave the while loop.\n      \tbreak;\n    }\n    catch (TransactionOptimisticException e) {\n    \t\t// Transaction has failed. Retry.\n    }\n}",
      "language": "java"
    }
  ]
}
[/block]
Another important point to note here is that a transaction will still fail even if an entry that was simply read (with no modify, cache.put(...)) since the value of the entry could be important to the logic within the initiated transaction.
[block:callout]
{
  "type": "info",
  "body": "In a highly concurrent environment, optimistic locking might lead to a high transaction failure rate but pessimistic locking can lead to deadlocks if locks are acquired in a different order by transactions."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Integration With JTA"
}
[/block]
Ignite can be configured with a JTA transaction manager lookup class using `TransactionConfiguration#setTxManagerLookupClassName` method. Transaction manager lookup is basically a factory that provides Ignite with an instance of JTA transaction manager.
When set, on each cache operation on a transactional cache Ignite will check if there is an ongoing JTA transaction. If JTA transaction is started, Ignite will also start a transaction and will enlist it into JTA transaction using it's own internal implementation of `XAResource`. Ignite transaction will be prepared, committed or rolledback altogether with corresponding JTA transaction.
Below is an example of using JTA transaction manager together with Ignite.
[block:code]
{
  "codes": [
    {
      "code": "// Get an instance of JTA transaction manager.\nTMService tms = appCtx.getComponent(TMService.class);\n\n// Get an instance of Ignite cache.\nIgniteCache<String, Integer> cache = cache();\n\nUserTransaction jtaTx = tms.getUserTransaction();\n\n// Start JTA transaction.\njtaTx.begin();\n\ntry {\n    // Do some cache operations.\n    cache.put(\"key1\", 1);\n    cache.put(\"key2\", 2);\n\n    // Commit the transaction.\n    jtaTx.commit();\n}\nfinally {\n    // Rollback in a case of exception.\n    if (jtaTx.getStatus() == Status.STATUS_ACTIVE)\n        jtaTx.rollback();\n}\n",
      "language": "java"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "Instead of creating a separate XA resource for each cache transaction, there is an option to enlist into JTA using lightweight synchronization callback (`javax.transaction.Synchronization`). In some cases this can give performance improvement, but keep in mind that most of the transaction managers do not allow to add more that one callback to a single transaction.\n\nTo enable this mode set `TransactionConfiguration#setUseJtaSynchronization` configuration flag to `true`.",
  "title": "Use javax.transaction.Synchronization"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Atomicity Mode"
}
[/block]
Ignite supports 2 atomicity modes defined in `CacheAtomicityMode` enum:
  * `TRANSACTIONAL`
  * `ATOMIC`

`TRANSACTIONAL` mode enables fully ACID-compliant transactions, however, when only atomic semantics are needed, it is recommended that  `ATOMIC` mode is used for better performance.

`ATOMIC` mode provides better performance by avoiding transactional locks, while still providing data atomicity and consistency. Another difference in `ATOMIC` mode is that bulk writes, such as `putAll(...)`and `removeAll(...)` methods are no longer executed in one transaction and can partially fail. In case of partial failure, `CachePartialUpdateException` will be thrown which will contain a list of keys for which the update failed.
[block:callout]
{
  "type": "info",
  "body": "Note that transactions are disabled whenever `ATOMIC` mode is used, which allows to achieve much higher performance and throughput in cases when transactions are not needed.",
  "title": "Performance"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
Atomicity mode is defined in `CacheAtomicityMode` enum and can be configured via `atomicityMode` property of `CacheConfiguration`. 

Default atomicity mode is `ATOMIC`.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          \t<!-- Set a cache name. -->\n   \t\t\t\t\t<property name=\"name\" value=\"myCache\"/>\n\n            <!-- Set atomicity mode, can be ATOMIC or TRANSACTIONAL. -->\n    \t\t\t\t<property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n            ... \n        </bean>\n    </property>\n          \n    <!-- Optional transaction configuration. -->\n    <property name=\"transactionConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.TransactionConfiguration\">\n            <!-- Configure TM lookup here. -->\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\ncacheCfg.setAtomicityMode(CacheAtomicityMode.ATOMIC);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Optional transaction configuration. Configure TM lookup here.\nTransactionConfiguration txCfg = new TransactionConfiguration();\n\ncfg.setTransactionConfiguration(txCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]