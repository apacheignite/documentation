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
      "language": "text"
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