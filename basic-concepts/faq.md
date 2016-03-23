- **What is the difference between on-heap and off-heap memory storage?**
Off-Heap memory allows your cache to overcome lengthy JVM Garbage Collection (GC) pauses when working with large heap sizes by caching data outside of main Java Heap space, but still in RAM.
[Read more](https://apacheignite.readme.io/docs/off-heap-memory)

- **Is Apache Ignite a key value / store?**
Apache Ignite is a resilient in-memory distributed object store with compute capabilities. In its simplest form, yes, Apache Ignite can be used as a key / value store (cache) but also exposes further rich APIs to interact with the data such as fully ANSI 99 compliant SQL querying, text searching, transactions etc.
[Read more](https://apacheignite.readme.io/docs/jcache)

- **Does Apache Ignite support JSON documents?**
Currently Apache Ignite does not fully support JSON documents at the present moment but the Node.JS client which is currently in beta will support JSON documents.

- **Can we use Apache Ignite with Apache Hive?**
Yes, Apache Ignite Hadoop Accelerator provides a set of components allowing for in-memory Hadoop job execution and file system operations for any Hadoop distribution including Apache Hive.
[Running Apache Hive over Ignited Hadoop](https://apacheignite-fs.readme.io/docs/running-apache-hive-over-ignited-hadoop)

- **In `PESSIMISTIC` Mode with transaction Isolation, do you lock keys for reading and writing?**
Yes, the main difference is that in `PESSIMISTIC` mode locks are acquired at the time of access, while in `OPTIMISTIC` mode locks are acquired during the commit phase.
[Read more](https://apacheignite.readme.io/docs/transactions)

- **Can I use Hibernate to access Apache Ignite?**
Yes, Apache Ignite In-Memory Data Fabric can be used as Hibernate Second-Level cache (or L2 cache), which can significantly speed-up the persistence layer of your application.
[Read more](https://apacheignite.readme.io/docs/hibernate-l2-cache)

- **Does Apache Ignite support JDBC?**
Yes, Apache Ignite is shipped with JDBC driver that allows you to retrieve distributed data from cache using standard SQL queries and JDBC API.
[Read more](https://apacheignite.readme.io/docs/jdbc-driver)

- **Does Apache Ignite guarantee ordering of messages?**
Yes, `sendOrdered(...)` method can be used if you want to receive messages in the order they were sent. A timeout parameter is passed to specify how long a message will stay in the queue to wait for messages that are supposed to be sent before this message. If the timeout expires, then all the messages that have not yet arrived for a given topic on that node will be ignored.
[Read more](https://apacheignite.readme.io/docs/messaging)

- **Can I run Java and .NET closures? How does that work?**
Java closures (compute jobs) can be executed only on Java Ignite nodes while .Net closures on .Net nodes only. So, if you decide to execute closures over server nodes then all the server nodes must be either Java nodes or .Net nodes depending on what kind of closures you're sending.

- **How do closures get shipped around?**
Every closure is an object of a particular class. When the closure is being sent it gets serialized to a binary form, send over the wire to a remote node and desirialised there.
The remote node should have the closure's class in its classpath or enable peerClassLoading in order to load the class from the sender side.

- **Are SQL queries load balanced?**
SQL queries are always broadcasted to every node that keeps data for caches used in a query. The exception is local SQL queries (query.setLocal(true) that are executed on a local node only.