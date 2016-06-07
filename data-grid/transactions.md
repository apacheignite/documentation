- [Atomicity Mode](doc:transactions#atomicity-mode)
- [IgniteTransactions](doc:transactions#ignitetransactions)
- [Two-Phase-Commit (2PC)](doc:transactions#two-phase-commit-2pc)
- [Concurrency Modes and Isolation Levels](doc:transactions#concurrency-modes-and-isolation-levels)
- [Pessimistic Transactions](doc:transactions#pessimistic-transactions) 
- [Optimistic Transactions](doc:transactions#optimistic-transactions) 
- [Deadlock Detection](doc:transactions#deadlock-detection) 
- [Deadlock-Free Transactions](doc:transactions#deadlock-free-transactions) 
- [Integration with JTA](doc:transactions#integration-with-jta) 
[block:api-header]
{
  "type": "basic",
  "title": "Atomicity Mode"
}
[/block]
Ignite supports 2 modes for cache operations, *transactional* and *atomic*. In `transactional` mode you are able to group multiple cache operations in a transaction, while `atomic` mode supports multiple atomic operations, one at a time.

These atomicity modes are defined in `CacheAtomicityMode` enum:
  * `TRANSACTIONAL`
  * `ATOMIC`
  
`TRANSACTIONAL` mode enables fully ACID-compliant transactions, however, when only atomic semantics are needed, it is recommended that  `ATOMIC` mode is used for better performance.

`ATOMIC` mode provides better performance by avoiding transactional locks, while still providing data atomicity and consistency. Another difference in `ATOMIC` mode is that bulk writes, such as `putAll(...)`and `removeAll(...)` methods are no longer executed in one transaction and can partially fail. In case of partial failure, `CachePartialUpdateException` will be thrown which will contain a list of keys for which the update failed.

Atomicity mode is defined in CacheAtomicityMode enum and can be configured via atomicityMode property of CacheConfiguration.
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    ...\n    <property name=\"cacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          \t<!-- Set a cache name. -->\n   \t\t\t\t\t<property name=\"name\" value=\"myCache\"/>\n\n            <!-- Set atomicity mode, can be ATOMIC or TRANSACTIONAL. \n\t\t\t\t\t\t\t\t ATOMIC is default. -->\n    \t\t\t\t<property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n            ... \n        </bean>\n    </property>\n          \n    <!-- Optional transaction configuration. -->\n    <property name=\"transactionConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.TransactionConfiguration\">\n            <!-- Configure TM lookup here. -->\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "CacheConfiguration cacheCfg = new CacheConfiguration();\n\ncacheCfg.setName(\"cacheName\");\n\ncacheCfg.setAtomicityMode(CacheAtomicityMode.ATOMIC);\n\nIgniteConfiguration cfg = new IgniteConfiguration();\n\ncfg.setCacheConfiguration(cacheCfg);\n\n// Optional transaction configuration. Configure TM lookup here.\nTransactionConfiguration txCfg = new TransactionConfiguration();\n\ncfg.setTransactionConfiguration(txCfg);\n\n// Start Ignite node.\nIgnition.start(cfg);",
      "language": "java"
    }
  ]
}
[/block]

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

[block:callout]
{
  "type": "info",
  "body": "Near caches are fully transactional and get updated or invalidated automatically whenever the data changes on the servers.",
  "title": "Near Cache Transactions"
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
  "title": "Concurrency Modes and Isolation Levels"
}
[/block]
Whenever `TRANSACTIONAL` atomicity mode is configured, Ignite supports `OPTIMISTIC` and `PESSIMISTIC` **concurrency modes** for transactions. Concurrency level determines when an entry-level transaction lock should be acquired - at the time of data access or during the `prepare` phase. Locking prevents concurrent access to an object. For example, when you attempt to update a ToDo list item with pessimistic locking, the server places a lock on the object until you either commit or rollback the transaction so that no other transaction or operation is allowed to update the same entry. Regardless of the concurrency level used in a transaction, there exists a moment in time when all entries enlisted in the transaction are locked before the commit.
**Isolation level** defines how concurrent transactions will 'see' and handle operations on the same keys. Ignite supports `READ_COMMITTED`, `REPEATABLE_READ` and `SERIALIZABLE` isolation levels. 
All combinations of concurrency modes and isolation levels can be used simultaneously. Below is the description of Ignite behavior and guarantees provided by each concurrency-isolation combination.
[block:api-header]
{
  "type": "basic",
  "title": "Pessimistic Transactions"
}
[/block]
In `PESSIMISTIC` transactions, locks are acquired during the first read or write access (depending on the isolation level) and held by the transaction until it is committed or rolled back. In this mode locks are acquired on primary nodes first and then promoted to backup nodes during the prepare stage. The following isolation levels can be configured with `PESSIMISTIC` concurrency mode:
  * `READ_COMMITTED`  - Data is read without a lock and is never cached in the transaction itself. The data may be read from a backup node if this is allowed in the cache configuration. In this isolation you can have the so-called Non-Repeatable Reads because a concurrent transaction can change the data when you are reading the data twice in your transaction. The lock is only acquired at the time of first write access (this includes `EntryProcessor` invocation). This means that an entry that have been read during the transaction may have a different value by the time the transaction is committed. No exception will be thrown in this case. 

  * `REPEATABLE_READ`  - Entry lock is acquired and data is fetched from the primary node on the first read or write access and stored in the local transactional map. All consecutive access to the same data is local and will return the last read or updated transaction value. This means no other concurrent transactions can make changes to the locked data, and you are getting Repeatable Reads for your transaction.

  * `SERIALIZABLE` - In `PESSIMISTIC` mode, this isolation level works the same way as `REPEATABLE_READ`. 

Note that in `PESSIMISTIC` mode, the order of locking is important. Moreover, Ignite will acquire locks sequentially and exactly in the order provided by a user.
[block:callout]
{
  "type": "warning",
  "body": "Imagine that you have 3 nodes in your topology (A, B, C) and in your transaction you are doing a `putAll` for keys [1, 2, 3, 4, 5, 6]. Suppose that these keys are mapped to nodes in the following fashion: {A: 1, 4}, {B: 2, 5}, {C: 3, 6}. Since Ignite cannot re-arrange the lock acquisition order in `PESSIMISTIC` mode, it will have to make 6 sequential network round-trips: [A, B, C, A, B, C]. In a case when the key locking order is not important for the semantics of a transaction, it is advisable to group keys by partition and lock keys within the same partition together. This may significantly reduce the number of network messages in a large transaction. In this example, if keys were ordered for a `putAll` in the following way: [1, 4, 2, 5, 3, 6], then only 3 sequential round-trips would be required.",
  "title": "Performance Considerations"
}
[/block]

[block:callout]
{
  "type": "danger",
  "body": "Note that if at least one PESSIMISTIC transaction lock is acquired, it will be impossible to change the cache topology until the transaction is committed or rolled back. Therefore, it is not recommended to hold transaction locks for a long period of time.",
  "title": "Topology Change Restrictions"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Optimistic Transactions"
}
[/block]
In `OPTIMISTIC` transactions, entry locks are acquired on primary nodes during the `prepare` step, then promoted to backup nodes and released once the transaction is committed. The locks are never acquired if the transaction is rolled back by user and no commit attempt was made. The following isolation levels can be configured with `OPTIMISTIC` concurrency mode:

 * `READ_COMMITTED` -  Changes that should be applied to the cache are collected on the originating node and applied upon the transaction commit. Transaction data is read without a lock and is never cached in the transaction. The data may be read from a backup node if this is allowed in the cache configuration. In this isolation you can have so-called Non-Repeatable Reads because a concurrent transaction can change the data when you are reading the data twice in your transaction. This mode combination does not check if the entry value has been modified since the first read or write access and never raises an optimistic exception.
 
 * `REPEATABLE_READ`  - Transactions at this isolation level work similar to `OPTIMISTIC` `READ_COMMITTED` transactions with only one difference - read values are cached on the originating node and all subsequent reads are guaranteed to be local. This mode combination does not check if the entry value has been modified since the first read or write access and never raises an optimistic exception.
 
 * `SERIALIZABLE`  - Stores an entry version upon first read access. Ignite will fail a transaction at the commit stage if the Ignite engine detects that at least one of the entries used as  part of the initiated transaction has been modified. This is achieved by internally checking the version of an entry remembered in a transaction to the one actually in the grid at the time of commit. In short, this means that if Ignite detects that there is a conflict at the commit stage of a transaction, we fail such a transaction throwing `TransactionOptimisticException` & rolling back any changes made. User should handle this exception and retry the transaction.
[block:code]
{
  "codes": [
    {
      "code": "IgniteTransactions txs = ignite.transactions();\n\n// Start transaction in optimistic mode with serializable isolation level.\nwhile (true) {\n    try (Transaction tx =  \n         ignite.transactions().txStart(TransactionConcurrency.OPTIMISTIC,\n                                       TransactionIsolation.SERIALIZABLE)) {\n\t \t\t\t// Modify cache entires as part of this transacation.\n  \t\t\t....\n        \n  \t\t\t// commit transaction.  \n  \t\t\ttx.commit();\n\n      \t// Transaction succeeded. Leave the while loop.\n      \tbreak;\n    }\n    catch (TransactionOptimisticException e) {\n    \t\t// Transaction has failed. Retry.\n    }\n}",
      "language": "java"
    }
  ]
}
[/block]
Another important point to note here is that a transaction will still fail even if an entry that was simply read (with no modify, cache.put(...)), since the value of the entry could be important to the logic within the initiated transaction.

Note that the key order is important for `READ_COMMITTED` and `REPEATABLE_READ` transactions since the locks are still acquired sequentially in these modes.

For `OPTIMISTIC` `SERIALIZABLE` transactions locks are not acquired sequentially. In this mode keys can be accessed in any order because transaction locks are acquired in parallel with an additional check allowing Ignite to avoid deadlocks. Refer to `Deadlock-Free Transactions` section below for more details.

[block:api-header]
{
  "type": "basic",
  "title": "Deadlock Detection"
}
[/block]
One major rule that anyone has to follow when working with distributed transactions is that locks for keys, participating in a transaction, must be acquired in the same order. Violating this rule may lead to a distributed deadlock.

Ignite does not avoid distributed deadlocks, but rather has a built-in functionality that makes it easier to debug and fix such situations.

[block:callout]
{
  "type": "warning",
  "body": "Presently the deadlock detection procedure is supported for pessimistic transactions only. Support of optimistic transaction will be available in the next Apache Ignite release."
}
[/block]
As shown in the code snippet below, a transaction has been started with a timeout. If the timeout expires, the deadlock detection procedure will try to find a possible deadlock that might have caused the timeout. When the timeout expires, `TransactionTimeoutException` is generated and propagated to the application code as the cause of `CacheException` regardless of a deadlock. However, if a deadlock is detected, the cause of the returned `TransactionTimeoutException` will be `TransactionDeadlockException` (at least for one transaction involved in the deadlock). 
[block:code]
{
  "codes": [
    {
      "code": "try (Transaction tx = ignite.transactions().txStart(TransactionConcurrency.PESSIMISTIC,\n    TransactionIsolation.READ_COMMITTED, 300, 0)) {\n    cache.put(1, 1);\n\n    cache.put(2, 1);\n\n    tx.commit();\n}\ncatch (CacheException e) {\n    if (e.getCause() instanceof TransactionTimeoutException &&\n        e.getCause().getCause() instanceof TransactionDeadlockException)    \n        \n        System.out.println(e.getCause().getCause().getMessage());\n}",
      "language": "java"
    }
  ]
}
[/block]
 `TransactionDeadlockException` message contains useful information that can help you find the reason for the deadlock.
[block:code]
{
  "codes": [
    {
      "code": "Deadlock detected:\n\nK1: TX1 holds lock, TX2 waits lock.\nK2: TX2 holds lock, TX1 waits lock.\n\nTransactions:\n\nTX1 [txId=GridCacheVersion [topVer=74949328, time=1463469328421, order=1463469326211, nodeOrder=1], nodeId=ad68354d-07b8-4be5-85bb-f5f2362fbb88, threadId=73]\nTX2 [txId=GridCacheVersion [topVer=74949328, time=1463469328421, order=1463469326210, nodeOrder=1], nodeId=ad68354d-07b8-4be5-85bb-f5f2362fbb88, threadId=74]\n\nKeys:\n\nK1 [key=1, cache=default]\nK2 [key=2, cache=default]",
      "language": "shell"
    }
  ]
}
[/block]
Deadlock detection is a multi step procedure that may take many iterations depending on the number of nodes in the cluster, keys, and transactions that are involved in a possible deadlock. A deadlock detection initiator is a node where a transaction was started and failed with a `TransactionTimeoutException`. This node will investigate if a deadlock has occurred, by exchanging requests/responses with other remote nodes, and prepare a deadlock related report that is provided with the `TransactionDeadlockException`. Each such message (request/response) is known as an iteration. 

Since a transaction is not rolled back until the deadlock detection procedure is completed, sometimes, it makes sense to tune the parameters (shown below), if you want to have a predictable time for a transaction's rollback. 

- `IgniteSystemProperties.IGNITE_TX_DEADLOCK_DETECTION_MAX_ITERS` - Specifies the maximum number of iterations for the deadlock detection procedure. If the value of this property is less than or equal to zero, the deadlock detection will be disabled (1000 by default);
- `IgniteSystemProperties.IGNITE_TX_DEADLOCK_DETECTION_TIMEOUT` - Specifies timeout for the deadlock detection mechanism (1 minute by default).

Note that if there are too few iterations, you may get an incomplete deadlock-report.
[block:callout]
{
  "type": "success",
  "title": "",
  "body": "If you want to avoid deadlocks at all, refer to Deadlock-free Transactions section below."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Deadlock-free Transactions"
}
[/block]
For `OPTIMISTIC` `SERIALIZABLE` transactions locks are not acquired sequentially. In this mode keys can be accessed in any order because transaction locks are acquired in parallel with an additional check allowing Ignite to avoid deadlocks.
We need to introduce some concepts in order to describe how lock `SERIALIZABLE` transactions work. Each transaction in Ignite is assigned a comparable version called `XidVersion`. Upon transaction commit each entry that is written in the transaction is assigned a new comparable version called `EntryVersion`. An `OPTIMISTIC` `SERIALIZABLE` transaction with version `XidVersionA` will fail with a `TransactionOptimisticException` if:
 * There is an ongoing `PESSIMISTIC` or non-serializable `OPTIMISTIC` transaction holding a lock on an entry of the `SERIALIZABLE` transaction.
 * There is another ongoing `OPTIMISTIC` `SERIALIZABLE` transaction with version `XidVersionB` such that `XidVersionB > XidVersionA` and this transaction holds a lock on an entry of the `SERIALIZABLE` transaction.
 * By the time the `OPTIMISTIC` `SERIALIZABLE` transaction acquires all required locks there exists an entry with the current version different from the observed version before commit.
[block:callout]
{
  "type": "info",
  "body": "In a highly concurrent environment, optimistic locking might lead to a high transaction failure rate but pessimistic locking can lead to deadlocks if locks are acquired in a different order by transactions. \nHowever, in a contention-free environment optimistic serializable locking may provide better performance for large transactions because the number of network trips depends only on the number of nodes that the transaction spans and does not depend on the number of keys in the transaction."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Integration With JTA"
}
[/block]
Ignite can be configured with a JTA transaction manager lookup class using `TransactionConfiguration#setTxManagerFactory` method. Transaction manager factory is a factory that provides Ignite with an instance of JTA transaction manager.

Ignite provides `CacheJndiTmFactory` factory.  It's out-of-the-box transaction manager factory implementation that is using JNDI names to find TM.

When set, on each cache operation on a transactional cache Ignite will check if there is an ongoing JTA transaction. If JTA transaction is started, Ignite will also start a transaction and will enlist it into JTA transaction using it's own internal implementation of `XAResource`. Ignite transaction will be prepared, committed or rolledback altogether with corresponding JTA transaction.
Below is an example of using JTA transaction manager together with Ignite.
[block:code]
{
  "codes": [
    {
      "code": "// Get an instance of JTA transaction manager.\nTMService tms = appCtx.getComponent(TMService.class);\n\n// Get an instance of Ignite cache.\nIgniteCache<String, Integer> cache = cache();\n\nUserTransaction jtaTx = tms.getUserTransaction();\n\n// Start JTA transaction.\njtaTx.begin();\n\ntry {\n    // Do some cache operations.\n    cache.put(\"key1\", 1);\n    cache.put(\"key2\", 2);\n\n    // Commit the transaction.\n    jtaTx.commit();\n}\nfinally {\n    // Rollback in a case of exception.\n    if (jtaTx.getStatus() == Status.STATUS_ACTIVE)\n        jtaTx.rollback();\n}",
      "language": "java"
    }
  ]
}
[/block]