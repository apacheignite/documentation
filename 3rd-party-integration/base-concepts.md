To setup Cassandra as a persistent store, you should set `CacheStoreFactory` for your Ignite cache to `org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory`.

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
  "type": "danger",
  "title": "Serializable",
  "body": "Current implementation of `org.apache.ignite.cache.store.cassandra.utils.datasource.DataSource` doesn't implement `Serializable`, thus for distributed Ignite clusters, `CassandraCacheStoreFactory` could only be setup through Spring XML file, but not from code."
}
[/block]
In the below section these two beans and their configuration settings will be described in details.
[block:api-header]
{
  "type": "basic",
  "title": "DataSourceBean"
}
[/block]
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
[block:api-header]
{
  "type": "basic",
  "title": "PersistenceSettingsBean"
}
[/block]
This bean stores all the details(keyspace, table, partition options, POJO fields mapping and etc...) of how objects (keys and values) should be persisted into Cassandra database.

Constructor of `org.apache.ignite.cache.store.cassandra.utils.persistence.KeyValuePersistenceSettings` allows to create such bean from a string which contains XML configuration document of specific structure (see below) or from the resource pointing to XML document.

Here is the generic example of a XML configuration document (**persistence descriptor**) which specifies how Ignite cache keys and values should be serialized/deserialized to/from Cassandra:
[block:code]
{
  "codes": [
    {
      "code": "<!--\nRoot container for persistence settings configuration.\n\nNote: required element\n\nAttributes:\n  1) keyspace [required] - specifies keyspace for Cassandra tables which should be used to store key/value pairs\n  2) table    [required] - specifies Cassandra table which should be used to store key/value pairs\n  3) ttl      [optional] - specifies expiration period for the table rows (in seconds)\n-->\n<persistence keyspace=\"my_keyspace\" table=\"my_table\" ttl=\"86400\">\n    <!--\n    Specifies Cassandra keyspace options which should be used to create provided keyspace if it doesn't exist.\n\n    Note: optional element\n    -->\n    <keyspaceOptions>\n        REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3}\n        AND DURABLE_WRITES = true\n    </keyspaceOptions>\n\n    <!--\n    Specifies Cassandra table options which should be used to create provided table if it doesn't exist.\n\n    Note: optional element\n    -->\n    <tableOptions>\n        comment = 'A most excellent and useful table'\n        AND read_repair_chance = 0.2\n    </tableOptions>\n\n    <!--\n    Specifies persistent settings for Ignite cache keys.\n\n    Note: required element\n\n    Attributes:\n      1) class      [required] - java class name for Ignite cache key\n      2) strategy   [required] - one of three possible persistent strategies:\n            a) PRIMITIVE - stores key value as is, by mapping it to Cassandra table column with corresponding type.\n                Should be used only for simple java types (int, long, String, double, Date) which could be mapped\n                to corresponding Cassadra types.\n            b) BLOB - stores key value as BLOB, by mapping it to Cassandra table column with blob type.\n                Could be used for any java object. Conversion of java object to BLOB is handled by \"serializer\"\n                which could be specified in serializer attribute (see below).\n            c) POJO - stores each field of an object as a column having corresponding type in Cassandra table.\n                Provides ability to utilize Cassandra secondary indexes for object fields.\n      3) serializer [optional] - specifies serializer class for BLOB strategy. Shouldn't be used for PRIMITIVE and\n        POJO strategies. Available implementations:\n            a) org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer - uses standard Java\n                serialization framework\n            b) org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer - uses Kryo\n                serialization framework\n      4) column     [optional] - specifies column name for PRIMITIVE and BLOB strategies where to store key value.\n        If not specified column having 'key' name will be used. Shouldn't be used for POJO strategy.\n    -->\n    <keyPersistence class=\"org.mycompany.MyKeyClass\" strategy=\"...\" serializer=\"...\" column=\"...\">\n        <!--\n        Specifies partition key fields if POJO strategy used.\n\n        Note: optional element, only required for POJO strategy in case you want to manually specify\n            POJO fields to Cassandra columns mapping, instead of relying on dynamic discovering of\n            POJO fields and mapping them to the same columns of Cassandra table.\n        -->\n        <partitionKey>\n            <!--\n             Specifies mapping from POJO field to Cassandra table column.\n\n             Note: required element\n\n             Attributes:\n               1) name   [required] - POJO field name\n               2) column [optional] - Cassandra table column name. If not specified lowercase\n                  POJO field name will be used.\n            -->\n            <field name=\"companyCode\" column=\"company\" />\n            ...\n            ...\n        </partitionKey>\n\n        <!--\n        Specifies cluster key fields if POJO strategy used.\n\n        Note: optional element, only required for POJO strategy in case you want to manually specify\n            POJO fields to Cassandra columns mapping, instead of relying on dynamic discovering of\n            POJO fields and mapping them to the same columns of Cassandra table.\n        -->\n        <clusterKey>\n            <!--\n             Specifies mapping from POJO field to Cassandra table column.\n\n             Note: required element\n\n             Attributes:\n               1) name   [required] - POJO field name\n               2) column [optional] - Cassandra table column name. If not specified lowercase\n                  POJO field name will be used.\n               3) sort   [optional] - specifies sort order (asc or desc)\n            -->\n            <field name=\"personNumber\" column=\"number\" sort=\"desc\"/>\n            ...\n            ...\n        </clusterKey>\n    </keyPersistence>\n\n    <!--\n    Specifies persistent settings for Ignite cache values.\n\n    Note: required element\n\n    Attributes:\n      1) class      [required] - java class name for Ignite cache value\n      2) strategy   [required] - one of three possible persistent strategies:\n            a) PRIMITIVE - stores key value as is, by mapping it to Cassandra table column with corresponding type.\n                Should be used only for simple java types (int, long, String, double, Date) which could be mapped\n                to corresponding Cassadra types.\n            b) BLOB - stores key value as BLOB, by mapping it to Cassandra table column with blob type.\n                Could be used for any java object. Conversion of java object to BLOB is handled by \"serializer\"\n                which could be specified in serializer attribute (see below).\n            c) POJO - stores each field of an object as a column having corresponding type in Cassandra table.\n                Provides ability to utilize Cassandra secondary indexes for object fields.\n      3) serializer [optional] - specifies serializer class for BLOB strategy. Shouldn't be used for PRIMITIVE and\n        POJO strategies. Available implementations:\n            a) org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer - uses standard Java\n                serialization framework\n            b) org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer - uses Kryo\n                serialization framework\n      4) column     [optional] - specifies column name for PRIMITIVE and BLOB strategies where to store value.\n        If not specified column having 'value' name will be used. Shouldn't be used for POJO strategy.\n    -->\n    <valuePersistence class=\"org.mycompany.MyValueClass\" strategy=\"...\" serializer=\"...\" column=\"\">\n        <!--\n         Specifies mapping from POJO field to Cassandra table column.\n\n         Note: required element\n\n         Attributes:\n           1) name         [required] - POJO field name\n           2) column       [optional] - Cassandra table column name. If not specified lowercase\n              POJO field name will be used.\n           3) static       [optional] - boolean flag which specifies that column is static withing a given partition\n           4) index        [optional] - boolean flag specifying that secondary index should be created for the field\n           5) indexClass   [optional] - custom index java class name if you want to use custom index\n           6) indexOptions [optional] - custom index options\n        -->\n        <field name=\"firstName\" column=\"first_name\" static=\"...\" index=\"...\" indexClass=\"...\" indexOptions=\"...\"/>\n        ...\n        ...\n    </valuePersistence>\n</persistence>",
      "language": "xml"
    }
  ]
}
[/block]
Below are provided all the details about persistence descriptor configuration and its elements:
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">persistence</div>"
}
[/block]

[block:callout]
{
  "type": "danger",
  "title": "Required Element",
  "body": "Root container for persistence settings configuration."
}
[/block]
| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**keyspace**      | <sup>yes | <sup>Keyspace for Cassandra tables which should be used to store key/value pairs. If keyspace doesn't exist it will be created (if specified Cassandra account has appropriate permissions). |
| <sup>**table**  | <sup>yes | <sup>Cassandra tables which should be used to store key/value pairs. If table doesn't exist it will be created (if specified Cassandra account has appropriate permissions).|
| <sup>**ttl** | <sup>no | <sup>Expiration period for the table rows (in seconds). Use this [link](http://docs.datastax.com/en/cql/3.1/cql/cql_using/use_expire_c.html) to read more about Cassandra ttl.|

In the next chapters you'll find what child elements could be placed inside persistence settings container.
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">keyspaceOptions</div>"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Optional element",
  "body": "Options to create Cassandra keyspace specified in the **keyspace** attribute of persistence settings container."
}
[/block]
Keyspace will be created only if it doesn't exist and if account used to connect to Cassandra has appropriate permissions.

Text specified in this XML element is just a chunk of [CREATE KEYSPACE](http://docs.datastax.com/en/cql/3.0/cql/cql_reference/create_keyspace_r.html) Cassandra DDL statement which goes after **WITH** keyword.
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">tableOptions</div>"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Optional Element",
  "body": "Options to create Cassandra table specified in the **table** attribute of persistence settings container."
}
[/block]
Table will be created only if it doesn't exist and if account used to connect to Cassandra has appropriate permissions.

Text specified in this XML element is just a chunk of [CREATE TABLE](http://docs.datastax.com/en/cql/3.0/cql/cql_reference/create_table_r.html) Cassandra DDL statement which goes after **WITH** keyword.
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">keyPersistence</div>"
}
[/block]

[block:callout]
{
  "type": "danger",
  "title": "Required element",
  "body": "Persistent settings for Ignite cache keys."
}
[/block]
These settings specify how key objects from Ignite cache should be stored/loaded to/from Cassandra table:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**class**      | <sup>yes | <sup>Java class name for Ignite cache keys. |
| <sup>**strategy**  | <sup>yes | <sup>Specifies one of three possible persistent strategies (see below) which controls how object should be persisted/loaded to/from Cassandra table.|
| <sup>**serializer** | <sup>no | <sup>Serializer class for BLOB strategy (see below for available implementations). Shouldn't be used for PRIMITIVE and POJO strategies.|
| <sup>**column** | <sup>no | <sup>Column name for PRIMITIVE and BLOB strategies where to store key. If not specified, column having 'key' name will be used. Attribute shouldn't be specified for POJO strategy.|

Persistence strategies:

| **Name**      | **Description**      |
| :-------------| :-------------|
| <sup>**PRIMITIVE**     | <sup>Stores object as is, by mapping it to Cassandra table column with corresponding type. Should be used only for simple java types (int, long, String, double, Date) which could be directly mapped to corresponding Cassadra types. Use this [link](http://docs.datastax.com/en/developer/java-driver/2.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) to figure out Java to Cassandra types mapping. |
| <sup>**BLOB**     | <sup>Stores object as BLOB, by mapping it to Cassandra table column with blob type. Could be used for any java object. Conversion of java object to BLOB is handled by "serializer" which could be specified in serializer attribute of **keyPersistence** container. |
| <sup>**POJO**     | <sup>Stores each field of an object as a column having corresponding type in Cassandra table. Provides ability to utilize Cassandra secondary indexes for object fields. Could be used only for POJO objects following Java Beans convention and having their fields of [simple java type which could be directly mapped to corresponding Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html).|

Available serializer implementations:

| **Class**      | **Description**      |
| :-------------| :-------------|
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer** | <sup>Uses standard Java serialization framework |
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer** | <sup>Uses Kryo serialization framework |

If you are using **PRIMITIVE** or **BLOB** persistence strategy you don't need to specify internal elements of `keyPersistence` tag, cause the idea of these two strategies is that the whole object should be persisted into one column of Cassandra table (which could be specified by `column` attribute).

If you are using **POJO** persistence strategy you have two option:
* Leave `keyPersistence` tag empty - in a such case, all the fields of POJO object class will be detected automatically using such rules:
  * Only fields having simple java types which could be directly mapped to [appropriate Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) will be detected.
  * Fields discovery mechanism takes into account `@QuerySqlField` annotation:
    * If `name` attribute is specified it will be used as a column name for Cassandra table. Otherwise field name in a lowercase will be used as a column name.
    * If `descending` attribute is specified for a field mapped to **cluster key** column, it will be used to set sort order for the column. 
  * Fields discovery mechanism takes into account `@AffinityKeyMapped` annotation. All the fields marked by this annotation will be treated as [partition key](http://docs.datastax.com/en/cql/3.0/cql/ddl/ddl_compound_keys_c.html) fields (in an order as they are declared in a class). All other fields will be treated as [cluster key](http://docs.datastax.com/en/cql/3.0/cql/ddl/ddl_compound_keys_c.html) fields.
  * If there are no fields annotated with `@AffinityKeyMapped` all the discovered fields will be treated as [partition key](http://docs.datastax.com/en/cql/3.0/cql/ddl/ddl_compound_keys_c.html) fields.

* Specify persistence details inside `keyPersistence` tag - in a such case, you have to specify **partition key** fields mapping to Cassandra table columns inside `partitionKey` tag. This tag is used just as a container for mapping settings and doesn't have any attributes. Optionally (if you are going to use cluster key) you can also specify **cluster key** fields mapping to appropriate Cassandra table columns inside `clusterKey` tag. This tag is used just as a container for mapping settings and doesn't have any attributes. 

Next two sections are providing detailed specification for **partition** and **cluster** key fields mappings (which makes sense if you choose second option from the list above).
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">partitionKey</div>"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Optional Element",
  "body": "Container for `field` elements specifying Cassandra partition key."
}
[/block]
Defines **Ignite cache** KEY object fields (inside it), which should be used as a **partition key** fields in Cassandra table and specifies fields mappings to table columns.

Mappings are specified by using `<field>` tag having such attributes:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**name**      | <sup>yes | <sup>POJO object field name. |
| <sup>**column**  | <sup>no | <sup>Cassandra table column name. If not specified lowercase POJO field name will be used.|
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">clusterKey</div>"
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Optional Element",
  "body": "Container for `field` elements specifying Cassandra cluster key."
}
[/block]
Defines **Ignite cache** KEY object fields (inside it), which should be used as a **cluster key** fields in Cassandra table and specifies fields mappings to table columns. 

Mapping are specified by using `<field>` tag having such attributes:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**name**      | <sup>yes | <sup>POJO object field name. |
| <sup>**column**  | <sup>no | <sup>Cassandra table column name. If not specified lowercase POJO field name will be used.|
| <sup>**sort**      | <sup>no | <sup>Specifies sort order for the field (**asc** or **desc**). |
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">valuePersistence</div>"
}
[/block]

[block:callout]
{
  "type": "danger",
  "title": "Required Element",
  "body": "Persistent settings for Ignite cache values."
}
[/block]
These settings specify how value objects from Ignite cache should be stored/loaded to/from Cassandra table. 

The settings attributes looks very similar to corresponding settings for Ignite cache keys:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**class**      | <sup>yes | <sup>Java class name for Ignite cache values. |
| <sup>**strategy**  | <sup>yes | <sup>Specifies one of three possible persistent strategies (see below) which controls how object should be persisted/loaded to/from Cassandra table.|
| <sup>**serializer** | <sup>no | <sup>Serializer class for BLOB strategy (see below for available implementations). Shouldn't be used for PRIMITIVE and POJO strategies.|
| <sup>**column** | <sup>no | <sup>Column name for PRIMITIVE and BLOB strategies where to store value. If not specified, column having 'value' name will be used. Attribute shouldn't be specified for POJO strategy.|

Persistence strategies (same as for key persistence settings):

| **Name**      | **Description**      |
| :-------------| :-------------|
| <sup>**PRIMITIVE**     | <sup>Stores object as is, by mapping it to Cassandra table column with corresponding type. Should be used only for simple java types (int, long, String, double, Date) which could be directly mapped to corresponding Cassadra types. Use this [link](http://docs.datastax.com/en/developer/java-driver/2.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) to figure out Java to Cassandra types mapping. |
| <sup>**BLOB**     | <sup>Stores object as BLOB, by mapping it to Cassandra table column with blob type. Could be used for any java object. Conversion of java object to BLOB is handled by "serializer" which could be specified in serializer attribute of **keyPersistence** container. |
| <sup>**POJO**     | <sup>Stores each field of an object as a column having corresponding type in Cassandra table. Provides ability to utilize Cassandra secondary indexes for object fields. Could be used only for POJO objects following Java Beans convention and having their fields of [simple java type which could be directly mapped to corresponding Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html).|

Available serializer implementations (same as for key persistence settings):

| **Class**      | **Description**      |
| :-------------| :-------------|
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.JavaSerializer** | <sup>Uses standard Java serialization framework |
| <sup>**org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer** | <sup>Uses Kryo serialization framework |

If you are using **PRIMITIVE** or **BLOB** persistence strategy you don't need to specify internal elements of `valuePersistence` tag, cause the idea of these two strategies is that the whole object should be persisted into one column of Cassandra table (which could be specified by `column` attribute).

If you are using **POJO** persistence strategy you have two option (similar to the same options for keys):
* Leave `valuePersistence` tag empty - in a such case, all the fields of POJO object class will be detected automatically using such rules:
  * Only fields having simple java types which could be directly mapped to [appropriate Cassandra types](http://docs.datastax.com/en/developer/java-driver/1.0/java-driver/reference/javaClass2Cql3Datatypes_r.html) will be detected.
  * Fields discovery mechanism takes into account `@QuerySqlField` annotation:
    * If `name` attribute is specified it will be used as a column name for Cassandra table. Otherwise field name in a lowercase will be used as a column name.
    * If `index` attribute is specified, secondary index will be created for corresponding column in Cassandra table (if such table does't exists). 
  
* Specify persistence details inside `valuePersistence` tag - in a such case, you have to specify your POJO fields mapping to Cassandra table columns inside `valuePersistence` tag.  

If you selected the second option from the list above, you have to use `<field>` tag to specify POJO fields to Cassandra table columns mapping. The tag has following attributes:

| **Attribute**      | **Required**      | **Description**          |
| :-------------| :-------------| :----------------|
| <sup>**name**      | <sup>yes | <sup>POJO object field name. |
| <sup>**column**  | <sup>no | <sup>Cassandra table column name. If not specified lowercase POJO field name will be used.|
| <sup>**static**  | <sup>no | <sup>Boolean flag which specifies that column is static withing a given partition.|
| <sup>**index**  | <sup>no | <sup>Boolean flag specifying that secondary index should be created for the field.|
| <sup>**indexClass**  | <sup>no | <sup>Custom index java class name, in case you want to use custom index.|
| <sup>**indexOptions**  | <sup>no | <sup>Custom index options.|