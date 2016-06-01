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

1. Utilizes [Apache Cassandra](http://cassandra.apache.org/), which is a very high performance and scalable key-value storage.
2. Uses Cassandra [asynchronous queries](http://www.datastax.com/dev/blog/java-driver-async-queries) for CacheStore batch operation [LOADALL(), WRITEALL(), DELETEALL()](http://apacheignite.gridgain.org/docs/persistent-store#section-loadall-writeall-deleteall-) to provide extremely high performance.
3.  Automatically creates all necessary tables (and keyspaces) in Cassandra if they are absent. Also automatically detects all the necessary fields for Ignite key-values which should be stored as POJO and creates appropriate table structure for you. Thus you don't need to care about Cassandra DDL syntax for table creation and Java to Cassandra type mapping details. You can also use `@QuerySqlField` annotation to provide configuration (column name, index, sort order) for Cassandra table columns.
4. You can optionally specify the settings (replication factor, replication strategy, bloom filter and etc.) for Cassandra tables and keyspaces which should be created.
5. Combines functionality of BLOB and POJO storage, allowing to specify how you prefer to store (as a BLOB or as a POJO) key-value pairs from your Ignite cache.
6. Supports Standard [Java](https://docs.oracle.com/javase/tutorial/jndi/objects/serial.html) and [Kryo](https://github.com/EsotericSoftware/kryo) serialization for key-values which should be stored as a BLOB in Cassandra
7. Supports Cassandra [secondary indexes](http://docs.datastax.com/en/cql/3.0/cql/cql_reference/create_index_r.html) (including custom indexes) through persistence configuration settings for particular Ignite cache or such settings could be detected automatically if you configured [SQL Indexes by Annotations](doc:sql-queries#configuring-sql-indexes-by-annotations) by using `@QuerySqlField(index = true)` annotation
8. Supports sort order for Cassandra cluster key fields through persistence configuration settings or such settings could be detected automatically if you are using `@QuerySqlField(descending = true)` annotation.
9. Supports [Affinity Collocation](doc:affinity-collocation) for the POJO key classes having one of their fields annotated by `@AffinityKeyMapped`. In a such way, key-values pairs which were stored on one node in Ignite cache will be also stored (collocated) on one node in Cassandra. 
[block:api-header]
{
  "type": "basic",
  "title": "Base concepts"
}
[/block]
To setup Cassandra as a persistent store you should set `CacheStoreFactory` for your Ignite cache to `org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory`.

This could be done using Spring context configuration like this:
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <property name=\"cacheConfiguration\">\n        <list>\n            ...\n            <!-- Configuring persistence for \"cache1\" cache -->\n            <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n                <property name=\"name\" value=\"cache1\"/>\n                <!-- Tune on Read-Through and Write-Through mode -->\n                <property name=\"readThrough\" value=\"true\"/>\n                <property name=\"writeThrough\" value=\"true\"/>\n                <!-- Specifying CacheStoreFactory -->\n                <property name=\"cacheStoreFactory\">\n                    <bean class=\"org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory\">\n                        <!-- Datasource configuration bean which is responsible for Cassandra connection details -->\n                        <property name=\"dataSourceBean\" value=\"cassandraDataSource\"/>\n                        <!-- Persistent settings bean which is responsible for the details of how objects will be persisted to Cassandra -->\n                        <property name=\"persistenceSettingsBean\" value=\"cache1_persistence_settings\"/>\n                    </bean>\n                </property>\n            </bean>\n            ...\n        </list>\n        ...\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
There are two main properties which should be specified for `CassandraCacheStoreFactory`:
- `dataSourceBean` - instance of `org.apache.ignite.cache.store.cassandra.utils.datasource.DataSource` class responsible for all the aspects of Cassandra database connection (credentials, contact points, read/write consistency level, load balancing policy and etc...)
- `persistenceSettingsBean` - instance of `org.apache.ignite.cache.store.cassandra.utils.persistence.KeyValuePersistenceSettings` class responsible for all the aspects of how objects should be persisted into Cassandra (keyspace and its options, table and its options, partition and cluster key options, POJO object fields mapping, secondary indexes, serializer for BLOB objects and etc...)
[block:callout]
{
  "type": "warning",
  "body": "Current implementation of `org.apache.ignite.cache.store.cassandra.utils.datasource.DataSource` doesn't implement `Serializable`, thus for distributed Ignite clusters, `CassandraCacheStoreFactory` could only be setup through Spring XML file, but not from code.",
  "title": "Serializable"
}
[/block]
In the below section these two beans and their configuration settings will be described in details.
#DataSourceBean
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