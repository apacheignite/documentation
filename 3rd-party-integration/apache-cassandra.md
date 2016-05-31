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
```xml
<bean id="ignite.cfg" class="org.apache.ignite.configuration.IgniteConfiguration">
    <property name="cacheConfiguration">
        <list>
            ...
            <!-- Configuring persistence for "cache1" cache -->
            <bean class="org.apache.ignite.configuration.CacheConfiguration">
                <property name="name" value="cache1"/>
                <!-- Tune on Read-Through and Write-Through mode -->
                <property name="readThrough" value="true"/>
                <property name="writeThrough" value="true"/>
                <!-- Specifying CacheStoreFactory -->
                <property name="cacheStoreFactory">
                    <bean class="org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory">
                        <!-- Datasource configuration bean which is responsible for Cassandra connection details -->
                        <property name="dataSourceBean" value="cassandraDataSource"/>
                        <!-- Persistent settings bean which is responsible for the details of how objects will be persisted to Cassandra -->
                        <property name="persistenceSettingsBean" value="cache1_persistence_settings"/>
                    </bean>
                </property>
            </bean>
            ...
        </list>
        ...
    </property>
</bean>

```

There are two main properties which should be specified for `CassandraCacheStoreFactory`:
- **dataSourceBean** - instance of `org.apache.ignite.cache.store.cassandra.utils.datasource.DataSource` class responsible for all the aspects of Cassandra database connection (credentials, contact points, read/write consistency level, load balancing policy and etc...)
- **persistenceSettingsBean** - instance of `org.apache.ignite.cache.store.cassandra.utils.persistence.KeyValuePersistenceSettings` class responsible for all the aspects of how objects should be persisted into Cassandra (keyspace and its options, table and its options, partition and cluster key options, POJO object fields mapping, secondary indexes, serializer for BLOB objects and etc...)

><sup>**Current implementation of `org.apache.ignite.cache.store.cassandra.utils.datasource.DataSource` doesn't implement `Serializable`, thus for distributed Ignite clusters, `CassandraCacheStoreFactory` could only be setup through Spring XML file, but not from code.**

In the below section these two beans and their configuration settings will be described in details.

#DataSourceBean

This bean stores all the details required for Cassandra database connection and CRUD operations. In the table below you can find all the bean properties:

| Property      | Default          | Description |
| :-------------| :----------------| :-----|
| <sup>**user**      |  | <sup>User name used to connect to Cassandra |
| <sup>**password**  |  |   <sup>User password used to connect to Cassandra |
| <sup>**credentials** |  | <sup>Credentials bean providing **username** and **password** |
| <sup>**authProvider** |  | <sup>Use the specified AuthProvider when connecting to Cassandra. Use this method when a custom authentication scheme is in place. |
| <sup>**port** |  | <sup>Port to use to connect to Cassandra (if it's not provided in connection point specification) |
| <sup>**contactPoints** |  | <sup>Array of contact points (**hostaname:[port]**) to use for Cassandra connection |
| <sup>**maxSchemaAgreementWaitSeconds** | <sup>10 sec | <sup>Maximum time to wait for schema agreement before returning from a DDL query |
| <sup>**protocolVersion** | <sup>3 | <sup>Specifies what version of Cassandra driver protocol should be used (could be helpful for backward compatibility with old versions of Cassandra) |
| <sup>**compression** |  | <sup>Compression to use for the transport. Supported compressions: **snappy**, **lz4** |
| <sup>**useSSL** | <sup>false | <sup>Enables the use of SSL |
| <sup>**sslOptions** | <sup>false | <sup>Enables the use of SSL using the provided options |
| <sup>**collectMetrix** | <sup>false | <sup>Enables metrics collection |
| <sup>**jmxReporting** | <sup>false | <sup>Enables JMX reporting of the metrics |
| <sup>**fetchSize** |  | <sup>Specifies query fetch size. Fetch size controls how much resulting rows will be retrieved simultaneously. |
| <sup>**readConsistency** |  | <sup>Specifies consistency level for READ queries |
| <sup>**writeConsistency** |  | <sup>Specifies consistency level for WRITE/DELETE/UPDATE queries |
| <sup>**loadBalancingPolicy** | <sup>TokenAwarePolicy | <sup>Specifies load balancing policy to use |
| <sup>**reconnectionPolicy** | <sup>ExponentialReconnectionPolicy | <sup>Specifies reconnection policy to use |
| <sup>**retryPolicy** | <sup>DefaultRetryPolicy | <sup>Specifies retry policy to use |
| <sup>**addressTranslater** | <sup>IdentityTranslater | <sup>Specifies address translater to use |
| <sup>**speculativeExecutionPolicy** | <sup>NoSpeculativeExecutionPolicy | <sup>Specifies speculative execution policy to use |
| <sup>**poolingOptions** |  | <sup>Specifies connection pooling options |
| <sup>**socketOptions** |  | <sup>Specifies low-level socket options for the connections kept to the Cassandra hosts |
| <sup>**nettyOptions** |  | <sup>Hooks that allow clients to customize Cassandra driver's underlying Netty layer |

#PersistenceSettingsBean

This bean stores all the details(keyspace, table, partition options, POJO fields mapping and etc...) of how objects (keys and values) should be persisted into Cassandra database.

Constructor of `org.apache.ignite.cache.store.cassandra.utils.persistence.KeyValuePersistenceSettings` allows to create such bean from a string which contains XML configuration document of specific structure (see below) or from the resource pointing to XML document.

Here is the generic example of a XML configuration document (persistence descriptor) which specifies how Ignite cache keys and values should be serialized/deserialized to/from Cassandra:

```xml
<!--
Root container for persistence settings configuration.

Note: required element

Attributes:
  1) keyspace [required] - specifies keyspace for Cassandra tables which should be used to store key/value pairs
  2) table    [required] - specifies Cassandra tables which should be used to store key/value pairs
  3) ttl      [optional] - specifies expiration period for the table rows (in seconds)
-->
<persistence keyspace="my_keyspace" table="my_table" ttl="86400">
    <!--
    Specifies Cassandra keyspace options which should be used to create provided keyspace if it doesn't exist.

    Note: optional element
    -->
    <keyspaceOptions>
        REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3}
        AND DURABLE_WRITES = true
    </keyspaceOptions>

    <!--
    Specifies Cassandra table options which should be used to create provided table if it doesn't exist.

    Note: optional element
    -->
    <tableOptions>
        comment = 'A most excellent and useful table'
        AND read_repair_chance = 0.2
    </tableOptions>

    <!--
    Specifies persistent settings for Ignite cache keys.

    Note: required element

    Attributes:
      1) class      [required] - java class name for Ignite cache key
      2) strategy   [required] - one of three possible persistent strategies:
            a) PRIMITIVE - stores key value as is, by mapping it to Cassandra table column with corresponding type.
                Should be used only for simple java types (int, long, String, double, Date) which could be mapped
                to corresponding Cassadra types.
            b) BLOB - stores key value as BLOB, by mapping it to Cassandra table column with blob type.
                Could be used for any java object. Conversion of java object to BLOB is handled by "serializer"
                which could be specified in serializer attribute (see below).
            c) POJO - stores each field of an object as a column having corresponding type in Cassandra table.
                Provides ability to utilize Cassandra secondary indexes for object fields.
      3) serializer [optional] - specifies serializer class for BLOB strategy. Shouldn't be used for PRIMITIVE and
        POJO strategies. Available implementations:
            a) org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer - uses standard Java
                serialization framework
            b) org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer - uses Kryo
                serialization framework
      4) column     [optional] - specifies column name for PRIMITIVE and BLOB strategies where to store key value.
        If not specified column having 'key' name will be used. Shouldn't be used for POJO strategy.
    -->
    <keyPersistence class="org.mycompany.MyKeyClass" strategy="..." serializer="..." column="...">
        <!--
        Specifies partition key fields if POJO strategy used.

        Note: optional element, only required for POJO strategy in case you want to manually specify
            POJO fields to Cassandra columns mapping, instead of relying on dynamic discovering of
            POJO fields and mapping them to the same columns of Cassandra table.
        -->
        <partitionKey>
            <!--
             Specifies mapping from POJO field to Cassandra table column.

             Note: required element

             Attributes:
               1) name   [required] - POJO field name
               2) column [optional] - Cassandra table column name. If not specified lowercase
                  POJO field name will be used.
            -->
            <field name="companyCode" column="company" />
            ...
            ...
        </partitionKey>

        <!--
        Specifies cluster key fields if POJO strategy used.

        Note: optional element, only required for POJO strategy in case you want to manually specify
            POJO fields to Cassandra columns mapping, instead of relying on dynamic discovering of
            POJO fields and mapping them to the same columns of Cassandra table.
        -->
        <clusterKey>
            <!--
             Specifies mapping from POJO field to Cassandra table column.

             Note: required element

             Attributes:
               1) name   [required] - POJO field name
               2) column [optional] - Cassandra table column name. If not specified lowercase
                  POJO field name will be used.
               3) sort   [optional] - specifies sort order (asc or desc)
            -->
            <field name="personNumber" column="number" sort="desc"/>
            ...
            ...
        </clusterKey>
    </keyPersistence>

    <!--
    Specifies persistent settings for Ignite cache values.

    Note: required element

    Attributes:
      1) class      [required] - java class name for Ignite cache value
      2) strategy   [required] - one of three possible persistent strategies:
            a) PRIMITIVE - stores key value as is, by mapping it to Cassandra table column with corresponding type.
                Should be used only for simple java types (int, long, String, double, Date) which could be mapped
                to corresponding Cassadra types.
            b) BLOB - stores key value as BLOB, by mapping it to Cassandra table column with blob type.
                Could be used for any java object. Conversion of java object to BLOB is handled by "serializer"
                which could be specified in serializer attribute (see below).
            c) POJO - stores each field of an object as a column having corresponding type in Cassandra table.
                Provides ability to utilize Cassandra secondary indexes for object fields.
      3) serializer [optional] - specifies serializer class for BLOB strategy. Shouldn't be used for PRIMITIVE and
        POJO strategies. Available implementations:
            a) org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer - uses standard Java
                serialization framework
            b) org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer - uses Kryo
                serialization framework
      4) column     [optional] - specifies column name for PRIMITIVE and BLOB strategies where to store value.
        If not specified column having 'value' name will be used. Shouldn't be used for POJO strategy.
    -->
    <valuePersistence class="org.mycompany.MyValueClass" strategy="..." serializer="..." column="">
        <!--
         Specifies mapping from POJO field to Cassandra table column.

         Note: required element

         Attributes:
           1) name         [required] - POJO field name
           2) column       [optional] - Cassandra table column name. If not specified lowercase
              POJO field name will be used.
           3) static       [optional] - boolean flag which specifies that column is static withing a given partition
           4) index        [optional] - boolean flag specifying that secondary index should be created for the field
           5) indexClass   [optional] - custom index java class name if you want to use custom index
           6) indexOptions [optional] - custom index options
        -->
        <field name="firstName" column="first_name" static="..." index="..." indexClass="..." indexOptions="..."/>
        ...
        ...
    </valuePersistence>
</persistence>
```

Below are provided all the details about persistence descriptor configuration and its elements:

## persistence 

<table><tr valign="middle"><td>:red_circle:</td><td style="font-size: 7pt;">Required element</td></tr></table>

Root container for persistence settings configuration.

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**keyspace**      | <sup>yes | <sup>Keyspace for Cassandra tables which should be used to store key/value pairs. If keyspace doesn't exist it will be created (if specified Cassandra account has appropriate permissions). |
| <sup>**table**  | <sup>yes | <sup>Cassandra tables which should be used to store key/value pairs. If table doesn't exist it will be created (if specified Cassandra account has appropriate permissions).|
| <sup>**ttl** | <sup>no | <sup>Expiration period for the table rows (in seconds). Use this [link](http://docs.datastax.com/en/cql/3.1/cql/cql_using/use_expire_c.html) to read more about Cassandra ttl.|

In the next chapters you'll find what child elements could be placed inside persistence settings container.

### keyspaceOptions

<table><tr valign="middle"><td>:large_blue_circle:</td><td style="font-size: 7pt;">Optional element</td></tr></table>

Cassandra keyspace options which should be used to create the keyspace specified in **keyspace** attribute of persistence settings container. Such a keyspace will be created only if it doesn't exist and if account used to connect to Cassandra has appropriate permissions.

Text specified in this XML element is just a chunk of [CREATE KEYSPACE](http://docs.datastax.com/en/cql/3.0/cql/cql_reference/create_keyspace_r.html) Cassandra DDL statement which goes after **WITH** keyword.

### tableOptions

<table><tr valign="middle"><td>:large_blue_circle:</td><td style="font-size: 7pt;">Optional element</td></tr></table>

Cassandra table options which should be used to create the table specified in **table** attribute of persistence settings container. Such a table will be created only if it doesn't exist and if account used to connect to Cassandra has appropriate permissions.

Text specified in this XML element is just a chunk of [CREATE TABLE](http://docs.datastax.com/en/cql/3.0/cql/cql_reference/create_table_r.html) Cassandra DDL statement which goes after **WITH** keyword.

### keyPersistence

<table><tr valign="middle"><td>:red_circle:</td><td style="font-size: 7pt;">Required element</td></tr></table>

Persistent settings for **Ignite cache** keys. These settings specify how key objects from Ignite cache should be stored/loaded to/from Cassandra table.

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**class**      | <sup>yes | <sup>Java class name for Ignite cache keys. |
| <sup>**strategy**  | <sup>yes | <sup>Specifies one of three possible persistent strategies (see below) which controls how object should be persisted/loaded to/from Cassandra table.|
| <sup>**serializer** | <sup>no | <sup>Serializer class for BLOB strategy (see below for available implementations). Shouldn't be used for PRIMITIVE and POJO strategies.|
| <sup>**column** | <sup>no | <sup>Column name for PRIMITIVE and BLOB strategies where to store key. If not specified, column having 'key' name will be used. Attribute shouldn't be specified for POJO strategy.|

Persistence strategies

| **Name**      | **Description**      |
| :-------------| :-------------|
| <sup>**PRIMITIVE**     | <sup>Stores object as is, by mapping it to Cassandra table column with corresponding type. Should be used only for simple java types (int, long, String, double, Date) which could be directly mapped to corresponding Cassadra types. Use this [link](http://docs.datastax.com/en/developer/java-driver/2.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) to figure out Java to Cassandra types mapping. |
| <sup>**BLOB**     | <sup>Stores object as BLOB, by mapping it to Cassandra table column with blob type. Could be used for any java object. Conversion of java object to BLOB is handled by "serializer" which could be specified in serializer attribute of **keyPersistence** container. |
| <sup>**POJO**     | <sup>Stores each field of an object as a column having corresponding type in Cassandra table. Provides ability to utilize Cassandra secondary indexes for object fields. Could be used only for POJO objects following Java Beans convention and having their fields of [simple java type which could be directly mapped to corresponding Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html).|

Available serializer implementations

| **Class**      | **Description**      |
| :-------------| :-------------|
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer** | <sup>Uses standard Java serialization framework |
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer** | <sup>Uses Kryo serialization framework |

If you are using PRIMITIVE or BLOB persistence strategy you don't need to specify internal elements of `keyPersistence` tag, cause the idea of these two strategies is that the whole object should be persisted into one column of Cassandra table (which could be specified by 'column' attribute).

If you are using POJO persistence strategy you have two option:
* **Leave 'keyPersistence' tag empty** - in a such case, all the fields of POJO object class will be detected automatically using such rules:
  * Only fields having simple java types which could be directly mapped to [appropriate Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) will be detected.
  * Fields discovery mechanism takes into account `@QuerySqlField` annotation:
    * If `name` attribute is specified it will be used as a column name for Cassandra table. Otherwise field name in a lowercase will be used as a column name.
    * If `descending` attribute is specified for a field mapped to **cluster key** column, it will be used to set sort order for the column. 
  * Fields discovery mechanism takes into account `@AffinityKeyMapped` annotation. All the fields marked by this annotation will be treated as [partition key](http://docs.datastax.com/en/cql/3.0/cql/ddl/ddl_compound_keys_c.html) fields (in an order as they are declared in a class). All other fields will be treated as [cluster key](http://docs.datastax.com/en/cql/3.0/cql/ddl/ddl_compound_keys_c.html) fields.
  * If there are no fields annotated with `@AffinityKeyMapped` all the discovered fields will be treated as [partition key](http://docs.datastax.com/en/cql/3.0/cql/ddl/ddl_compound_keys_c.html) fields.

* **Specify persistence details inside 'keyPersistence' tag** - in a such case, you have to specify **partition key** fields mapping to Cassandra table columns inside `partitionKey` tag. This tag is used just as a container for mapping settings and doesn't have any attributes. Optionally (if you are going to use cluster key) you can also specify **cluster key** fields mapping to appropriate Cassandra table columns inside `clusterKey` tag. This tag is used just as a container for mapping settings and doesn't have any attributes. 

Next two sections are providing detailed specification for **partition** and **cluster** key fields mappings (which makes sense if you choose second option from the list above).

#### partitionKey

<table><tr valign="middle"><td>:large_blue_circle:</td><td style="font-size: 7pt;">Optional element</td></tr></table>

Used just as a container to specify **Ignite cache** KEY object fields, which should be used as a **partition key** fields in Cassandra table and specifies fields mappings to table columns. 

Mapping are specified by using `<field>` tag having such attributes:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**name**      | <sup>yes | <sup>POJO object field name. |
| <sup>**column**  | <sup>no | <sup>Cassandra table column name. If not specified lowercase POJO field name will be used.|

#### clusterKey

<table><tr valign="middle"><td>:large_blue_circle:</td><td style="font-size: 7pt;">Optional element</td></tr></table>

Used just as a container to specify **Ignite cache** KEY object fields, which should be used as a **cluster key** fields in Cassandra table and specifies fields mappings to table columns. 

Mapping are specified by using `<field>` tag having such attributes:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**name**      | <sup>yes | <sup>POJO object field name. |
| <sup>**column**  | <sup>no | <sup>Cassandra table column name. If not specified lowercase POJO field name will be used.|
| <sup>**sort**      | <sup>no | <sup>Specifies sort order for the field (**asc** or **desc**). |

### valuePersistence

<table><tr valign="middle"><td>:red_circle:</td><td style="font-size: 7pt;">Required element</td></tr></table>

Persistent settings for **Ignite cache** values. These settings specify how value objects from Ignite cache should be stored/loaded to/from Cassandra table and looks very similar to corresponding settings for **Ignite cache** keys.

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**class**      | <sup>yes | <sup>Java class name for Ignite cache values. |
| <sup>**strategy**  | <sup>yes | <sup>Specifies one of three possible persistent strategies (see below) which controls how object should be persisted/loaded to/from Cassandra table.|
| <sup>**serializer** | <sup>no | <sup>Serializer class for BLOB strategy (see below for available implementations). Shouldn't be used for PRIMITIVE and POJO strategies.|
| <sup>**column** | <sup>no | <sup>Column name for PRIMITIVE and BLOB strategies where to store value. If not specified, column having 'value' name will be used. Attribute shouldn't be specified for POJO strategy.|

Persistence strategies

| **Name**      | **Description**      |
| :-------------| :-------------|
| <sup>**PRIMITIVE**     | <sup>Stores object as is, by mapping it to Cassandra table column with corresponding type. Should be used only for simple java types (int, long, String, double, Date) which could be directly mapped to corresponding Cassadra types. Use this [link](http://docs.datastax.com/en/developer/java-driver/2.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) to figure out Java to Cassandra types mapping. |
| <sup>**BLOB**     | <sup>Stores object as BLOB, by mapping it to Cassandra table column with blob type. Could be used for any java object. Conversion of java object to BLOB is handled by "serializer" which could be specified in serializer attribute of **keyPersistence** container. |
| <sup>**POJO**     | <sup>Stores each field of an object as a column having corresponding type in Cassandra table. Provides ability to utilize Cassandra secondary indexes for object fields. Could be used only for POJO objects following Java Beans convention and having their fields of [simple java type which could be directly mapped to corresponding Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html).|

Available serializer implementations

| **Class**      | **Description**      |
| :-------------| :-------------|
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer** | <sup>Uses standard Java serialization framework |
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer** | <sup>Uses Kryo serialization framework |

If you are using PRIMITIVE or BLOB persistence strategy you don't need to specify internal elements of `valuePersistence` tag, cause the idea of these two strategies is that the whole object should be persisted into one column of Cassandra table (which could be specified by 'column' attribute).

If you are using POJO persistence strategy you have two option (similar to two options for keys):
* **Leave 'valuePersistence' tag empty** - in a such case, all the fields of POJO object class will be detected automatically using such rules:
  * Only fields having simple java types which could be directly mapped to [appropriate Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) will be detected.
  * Fields discovery mechanism takes into account `@QuerySqlField` annotation:
    * If `name` attribute is specified it will be used as a column name for Cassandra table. Otherwise field name in a lowercase will be used as a column name.
    * If `index` attribute is specified, secondary index will be created for corresponding column in Cassandra table (if such table does't exists). 
  
* **Specify persistence details inside 'valuePersistence' tag** - in a such case, you have to specify your POJO fields mapping to Cassandra table columns inside `valuePersistence` tag.  

If you selected the second option from the list above, you have to use `<field>` tag to specify POJO fields to Cassandra table columns mapping. The tag has following attributes:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**name**      | <sup>yes | <sup>POJO object field name. |
| <sup>**column**  | <sup>no | <sup>Cassandra table column name. If not specified lowercase POJO field name will be used.|
| <sup>**static**  | <sup>no | <sup>Boolean flag which specifies that column is static withing a given partition.|
| <sup>**index**  | <sup>no | <sup>Boolean flag specifying that secondary index should be created for the field.|
| <sup>**indexClass**  | <sup>no | <sup>Custom index java class name, in case you want to use custom index.|
| <sup>**indexOptions**  | <sup>no | <sup>Custom index options.|
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