As described in [Base concepts](doc:base-concepts) to configure Cassandra as a cache store you need to set **CacheStoreFactory** for your Ignite cache to `org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory`.

Below is an example of typical configuration for Ignite cache to use Cassandra as a cache store. The example is taken from the unit tests resource file `test/resources/org/apache/ignite/tests/persistence/blob/ignite-config.xml` of Cassandra module source code.
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\">\n\n    <!-- Cassandra connection settings -->\n    <import resource=\"classpath:org/apache/ignite/tests/cassandra/connection-settings.xml\" />\n\n    <!-- Persistence settings for 'cache1' -->\n    <bean id=\"cache1_persistence_settings\" class=\"org.apache.ignite.cache.store.cassandra.utils.persistence.KeyValuePersistenceSettings\">\n        <constructor-arg type=\"org.springframework.core.io.Resource\" value=\"classpath:org/apache/ignite/tests/persistence/blob/persistence-settings-1.xml\" />\n    </bean>\n\n    <!-- Persistence settings for 'cache2' -->\n    <bean id=\"cache2_persistence_settings\" class=\"org.apache.ignite.cache.store.cassandra.utils.persistence.KeyValuePersistenceSettings\">\n        <constructor-arg type=\"org.springframework.core.io.Resource\" value=\"classpath:org/apache/ignite/tests/persistence/blob/persistence-settings-3.xml\" />\n    </bean>\n\n    <!-- Ignite configuration -->\n    <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n        <property name=\"cacheConfiguration\">\n            <list>\n                <!-- Configuring persistence for \"cache1\" cache -->\n                <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n                    <property name=\"name\" value=\"cache1\"/>\n                    <property name=\"readThrough\" value=\"true\"/>\n                    <property name=\"writeThrough\" value=\"true\"/>\n                    <property name=\"cacheStoreFactory\">\n                        <bean class=\"org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory\">\n                            <property name=\"dataSourceBean\" value=\"cassandraAdminDataSource\"/>\n                            <property name=\"persistenceSettingsBean\" value=\"cache1_persistence_settings\"/>\n                        </bean>\n                    </property>\n                </bean>\n\n                <!-- Configuring persistence for \"cache2\" cache -->\n                <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n                    <property name=\"name\" value=\"cache2\"/>\n                    <property name=\"readThrough\" value=\"true\"/>\n                    <property name=\"writeThrough\" value=\"true\"/>\n                    <property name=\"cacheStoreFactory\">\n                        <bean class=\"org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory\">\n                            <property name=\"dataSourceBean\" value=\"cassandraAdminDataSource\"/>\n                            <property name=\"persistenceSettingsBean\" value=\"cache2_persistence_settings\"/>\n                        </bean>\n                    </property>\n                </bean>\n            </list>\n        </property>\n\n        <!-- Explicitly configure TCP discovery SPI to provide list of initial nodes. -->\n        <property name=\"discoverySpi\">\n            <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n                <property name=\"ipFinder\">\n                    <!--\n                        Ignite provides several options for automatic discovery that can be used\n                        instead os static IP based discovery. For information on all options refer\n                        to our documentation: http://apacheignite.readme.io/docs/cluster-config\n                    -->\n                    <!-- Uncomment static IP finder to enable static-based discovery of initial nodes. -->\n                    <!--<bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.vm.TcpDiscoveryVmIpFinder\">-->\n                    <bean class=\"org.apache.ignite.spi.discovery.tcp.ipfinder.multicast.TcpDiscoveryMulticastIpFinder\">\n                        <property name=\"addresses\">\n                            <list>\n                                <!-- In distributed environment, replace with actual host IP address. -->\n                                <value>127.0.0.1:47500..47509</value>\n                            </list>\n                        </property>\n                    </bean>\n                </property>\n            </bean>\n        </property>\n    </bean>\n</beans>",
      "language": "xml"
    }
  ]
}
[/block]
In the specified example we have two Ignite caches configured: `cache1` and `cache2`. So lets look at the configuration details.

Lets start from the cache configuration details. They are pretty similar for both caches (**cache1** and **cache2**) and looks like that:
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"cache1\"/>\n    <property name=\"readThrough\" value=\"true\"/>\n    <property name=\"writeThrough\" value=\"true\"/>\n    <property name=\"cacheStoreFactory\">\n        <bean class=\"org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory\">\n            <property name=\"dataSourceBean\" value=\"cassandraAdminDataSource\"/>\n            <property name=\"persistenceSettingsBean\" value=\"cache1_persistence_settings\"/>\n        </bean>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
First of all we can see that [read-through and write-through](doc:persistent-store#read-through-and-write-through) options are enabled:
[block:code]
{
  "codes": [
    {
      "code": "<property name=\"readThrough\" value=\"true\"/>\n<property name=\"writeThrough\" value=\"true\"/>",
      "language": "xml"
    }
  ]
}
[/block]
which is required for Ignite cache, if you plan to use persistent store for cache entries which expired.

You can optionally specify [write-behind](doc:persistent-store#write-behind-caching) setting if you prefer persistent store to be updated asynchronously:
[block:code]
{
  "codes": [
    {
      "code": "<property name=\"writeBehindEnabled\" value=\"true\"/>",
      "language": "xml"
    }
  ]
}
[/block]
The next important thing is `CacheStoreFactory` configuration:
[block:code]
{
  "codes": [
    {
      "code": "<property name=\"cacheStoreFactory\">\n    <bean class=\"org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory\">\n        <property name=\"dataSourceBean\" value=\"cassandraAdminDataSource\"/>\n        <property name=\"persistenceSettingsBean\" value=\"cache1_persistence_settings\"/>\n    </bean>\n</property>",
      "language": "xml"
    }
  ]
}
[/block]
You should use `org.apache.ignite.cache.store.cassandra.CassandraCacheStoreFactory` as a `CacheStoreFactory` for your Ignite cache to utilize Cassandra as a persistent store. For `CassandraCacheStoreFactory` you should specify two required properties:
* **dataSourceBean** - name of the Spring bean, which specifies all the details about Cassandra database connection. For more details visit this [link](doc:base-concepts#datasourcebean). 

* **persistenceSettingsBean** - name of the Spring bean, which specifies all the details about how objects should be persisted into Cassandra database. For more details visit this [link](doc:base-concepts#persistencesettingsbean).

In the specified example `cassandraAdminDataSource` is a data source bean, which is imported into Ignite cache config file using this directive: 
[block:code]
{
  "codes": [
    {
      "code": "<import resource=\"classpath:org/apache/ignite/tests/cassandra/connection-settings.xml\" />",
      "language": "xml"
    }
  ]
}
[/block]
and `cache1_persistence_settings` is a persistence settings bean, which is defined in Ignite cache config file using such directive:
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"cache1_persistence_settings\" class=\"org.apache.ignite.cache.store.cassandra.utils.persistence.KeyValuePersistenceSettings\">\n    <constructor-arg type=\"org.springframework.core.io.Resource\" value=\"classpath:org/apache/ignite/tests/persistence/blob/persistence-settings-1.xml\" />\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]
Now lets look at the specification of `cassandraAdminDataSource` from `org/apache/ignite/tests/cassandra/connection-settings.xml` test resource:
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\">\n\n    <bean id=\"cassandraAdminCredentials\" class=\"org.apache.ignite.tests.utils.CassandraAdminCredentials\"/>\n\n    <bean id=\"loadBalancingPolicy\" class=\"com.datastax.driver.core.policies.RoundRobinPolicy\"/>\n\n    <bean id=\"contactPoints\" class=\"org.apache.ignite.tests.utils.CassandraHelper\" factory-method=\"getContactPointsArray\"/>\n\n    <bean id=\"cassandraAdminDataSource\" class=\"org.apache.ignite.cache.store.cassandra.utils.datasource.DataSource\">\n        <property name=\"credentials\" ref=\"cassandraAdminCredentials\"/>\n        <property name=\"contactPoints\" ref=\"contactPoints\"/>\n        <property name=\"readConsistency\" value=\"ONE\"/>\n        <property name=\"writeConsistency\" value=\"ONE\"/>\n        <property name=\"loadBalancingPolicy\" ref=\"loadBalancingPolicy\"/>\n    </bean>\n</beans>",
      "language": "xml"
    }
  ]
}
[/block]
For more details about Cassandra data source connection configuration visit this [link](doc:base-concepts#datasourcebean) from [Base concepts](doc:base-concepts) page.

Finally, the last piece which wasn't still described is persistence settings configuration. Lets look at the `cache1_persistence_settings` from the `org/apache/ignite/tests/persistence/blob/persistence-settings-1.xml` test resource.
[block:code]
{
  "codes": [
    {
      "code": "<persistence keyspace=\"test1\" table=\"blob_test1\">\n    <keyPersistence class=\"java.lang.Integer\" strategy=\"PRIMITIVE\" />\n    <valuePersistence strategy=\"BLOB\"/>\n</persistence>",
      "language": "xml"
    }
  ]
}
[/block]
In the configuration above, we can see that Cassandra `test1.blob_test1` table will be used to store key/value objects for **cache1** cache. Key objects of the cache will be stored as **integer** in `key` column. Value objects of the cache will be stored as **blob** in `value` column. For more information about persistence settings configuration visit this [link](doc:base-concepts#persistencesettingsbean) from [Base concepts](doc:base-concepts) page.

Next sections will provide examples of persistence settings configuration for different kind of persistence strategies (see more details about persistence strategies in [Base concepts](doc:base-concepts).
[block:api-header]
{
  "type": "basic",
  "title": "Example 1"
}
[/block]
Persistence setting for Ignite cache with keys of `Integer` type to be persisted as `int` in Cassandra and values of `String` type to be persisted as `text` in Cassandra.
[block:code]
{
  "codes": [
    {
      "code": "<persistence keyspace=\"test1\" table=\"my_table\">\n    <keyPersistence class=\"java.lang.Integer\" strategy=\"PRIMITIVE\" column=\"my_key\"/>\n    <valuePersistence class=\"java.lang.String\" strategy=\"PRIMITIVE\" />\n</persistence>",
      "language": "xml"
    }
  ]
}
[/block]
Keys will be stored in `my_key` column. Values will be stored in `value` column (which is used by default if `column` attribute wasn't specified).
[block:api-header]
{
  "type": "basic",
  "title": "Example 2"
}
[/block]
Persistence setting for Ignite cache with keys of `Integer` type to be persisted as `int` in Cassandra and values of `any` type (you don't need to specify the type for **BLOB** persistence strategy) to be persisted as `blob` in Cassandra. The only solution for this situation is to store value as a `BLOB` in Cassandra table.
[block:code]
{
  "codes": [
    {
      "code": "<persistence keyspace=\"test1\" table=\"my_table\">\n    <keyPersistence class=\"java.lang.Integer\" strategy=\"PRIMITIVE\" />\n    <valuePersistence strategy=\"BLOB\"/>\n</persistence>",
      "language": "xml"
    }
  ]
}
[/block]
Keys will be stored in `key` column (which is used by default if `column` attribute wasn't specified). Values will be stored in `value` column.
[block:api-header]
{
  "type": "basic",
  "title": "Example 3"
}
[/block]
Persistence setting for Ignite cache with keys of `Integer` type and values of **any** type, both to be persisted as `BLOB` in Cassandra.
[block:code]
{
  "codes": [
    {
      "code": "<persistence keyspace=\"test1\" table=\"my_table\">\n    <!-- By default Java standard serialization is going to be used -->\n    <keyPersistence class=\"java.lang.Integer\"\n                    strategy=\"BLOB\"/>\n\n    <!-- Kryo serialization specified to be used -->\n    <valuePersistence class=\"org.apache.ignite.tests.pojos.Person\"\n                      strategy=\"BLOB\"\n                      serializer=\"org.apache.ignite.cache.store.cassandra.utils.serializer.KryoSerializer\"/>\n</persistence>",
      "language": "xml"
    }
  ]
}
[/block]
Keys will be stored in `key` column having `blob` type and using [Java standard serialization](https://docs.oracle.com/javase/tutorial/jndi/objects/serial.html). Values will be stored in `value` column having 'blob' type and using [Kryo serialization](https://github.com/EsotericSoftware/kryo).
[block:api-header]
{
  "type": "basic",
  "title": "Example 4"
}
[/block]
Persistence setting for Ignite cache with keys of `Integer` type to be persisted as `int` in Cassandra and values of custom POJO `org.apache.ignite.tests.pojos.Person` type to be dynamically analyzed and persisted into a set of table columns, so that each POJO field will be mapped to appropriate table column. For more details about dynamic POJO fields discovery read [this](doc:base-concepts#persistencesettingsbean) section from [Base concepts](doc:base-concepts).
[block:code]
{
  "codes": [
    {
      "code": "<persistence keyspace=\"test1\" table=\"my_table\">\n    <keyPersistence class=\"java.lang.Integer\" strategy=\"PRIMITIVE\"/>\n    <valuePersistence class=\"org.apache.ignite.tests.pojos.Person\" strategy=\"POJO\"/>\n</persistence>",
      "language": "xml"
    }
  ]
}
[/block]
Keys will be stored in `key` column having `int` type. 

Now lets imagine that `org.apache.ignite.tests.pojos.Person` class has such an implementation:
[block:code]
{
  "codes": [
    {
      "code": "public class Person {\n    private String firstName;\n    private String lastName;\n    private int age;\n    private boolean married;\n    private long height;\n    private float weight;\n    private Date birthDate;\n    private List<String> phones;\n\n    public void setFirstName(String name) {\n        firstName = name;\n    }\n\n    public String getFirstName() {\n        return firstName;\n    }\n\n    public void setLastName(String name) {\n        lastName = name;\n    }\n\n    public String getLastName() {\n        return lastName;\n    }\n\n    public void setAge(int age) {\n        this.age = age;\n    }\n\n    public int getAge() {\n        return age;\n    }\n\n    public void setMarried(boolean married) {\n        this.married = married;\n    }\n\n    public boolean getMarried() {\n        return married;\n    }\n\n    public void setHeight(long height) {\n        this.height = height;\n    }\n\n    public long getHeight() {\n        return height;\n    }\n\n    public void setWeight(float weight) {\n        this.weight = weight;\n    }\n\n    public float getWeight() {\n        return weight;\n    }\n\n    public void setBirthDate(Date date) {\n        birthDate = date;\n    }\n\n    public Date getBirthDate() {\n        return birthDate;\n    }\n\n    public void setPhones(List<String> phones) {\n        this.phones = phones;\n    }\n\n    public List<String> getPhones() {\n        return phones;\n    }\n}",
      "language": "java"
    }
  ]
}
[/block]
In this case Ignite cache values of `org.apache.ignite.tests.pojos.Person` type will be persisted into a set of Cassandra table columns using such dynamically configured mapping rule:

| POJO field    | Table column     | Column type |
| :-------------| :----------------| :----------|
| firstName     | firstname        | text    |
| lastName      | lastname         | text    |
| age           | age              | int     |
| married       | married          | boolean |
| height        | height           | bigint    |
| weight        | weight           | float    |
| birthDate     | birthdate        | timestamp    |