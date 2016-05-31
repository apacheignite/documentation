**Index**
* [Overview](#overview)
* [Base concepts](#base-concepts)
* [DDL generator](#ddl-generator)
* [Examples](#examples)
* [Load tests](#load-tests)
* [Unit tests](#unit-tests)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite Cassandra module implements [persistent store](doc:persistent-store) for Ignite caches by utilizing Cassandra as a persistent storage for expired cache records.

It functions pretty much the same way like [CacheJdbcBlobStore](doc:persistent-store#cachejdbcblobstore) and [CacheJdbcPojoStore](doc:persistent-store#cachejdbcpojostore) and provides such benefits:

  * Utilizes Apache Cassandra, which is a very high performance and scalable key-value storage.
  * Uses Cassandra asynchronous queries for CacheStore batch operation LOADALL(), WRITEALL(), DELETEALL() to provide extremely high performance.
  * Automatically creates all necessary tables (and keyspaces) in Cassandra if they are absent. Also automatically detects all the necessary fields for Ignite key-values which should be stored as POJO and creates appropriate table structure for you. Thus you don't need to care about Cassandra DDL syntax for table creation and Java to Cassandra type mapping details. You can also use @QuerySqlField annotation to provide configuration (column name, index, sort order) for Cassandra table columns.
  * You can optionally specify the settings (replication factor, replication strategy, bloom filter and etc.) for Cassandra tables and keyspaces which should be created.
  * Combines functionality of BLOB and POJO storage, allowing to specify how you prefer to store (as a BLOB or as a POJO) key-value pairs from your Ignite cache.
  * Supports Standard Java and Kryo serialization for key-values which should be stored as a BLOB in Cassandra
  * Supports Cassandra secondary indexes (including custom indexes) through persistence configuration settings for particular Ignite cache or such settings could be detected automatically if you configured SQL Indexes by Annotations by using @QuerySqlField(index = true) annotation
  * Supports sort order for Cassandra cluster key fields through persistence configuration settings or such settings could be detected automatically if you are using @QuerySqlField(descending = true) annotation.
  * Supports Affinity Collocation for the POJO key classes having one of their fields annotated by @AffinityKeyMapped. In a such way, key-values pairs which were stored on one node in Ignite cache will be also stored (collocated) on one node in Cassandra. 
[block:api-header]
{
  "type": "basic",
  "title": "Base concepts"
}
[/block]
[TBD]
[block:api-header]
{
  "type": "basic",
  "title": "DDL generator"
}
[/block]
[TBD]
[block:api-header]
{
  "type": "basic",
  "title": "Examples"
}
[/block]
[TBD]
[block:api-header]
{
  "type": "basic",
  "title": "Load tests"
}
[/block]
[TBD]
[block:api-header]
{
  "type": "basic",
  "title": "Unit tests"
}
[/block]
[TBD]