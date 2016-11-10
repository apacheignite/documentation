Starting from Ignite 1.8, DML is supported in SQL queries. This means DML can now be used in ODBC as well. On this you can find details.
[block:api-header]
{
  "type": "basic",
  "title": "Preparing node"
}
[/block]
With DML you can now insert, modify and delete records using SLQ alone, which means you can now manipulate data on Ignite writing virtually zero lines of Java code. Let me give you an example. Consider the following simple node configuration:
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\n        http://www.springframework.org/schema/util\n        http://www.springframework.org/schema/util/spring-util.xsd\">\n  <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <!-- Enabling ODBC. -->\n    <property name=\"odbcConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.OdbcConfiguration\"/>\n    </property>\n\n    <!-- Configuring cache. -->\n    <property name=\"cacheConfiguration\">\n      <list>\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Person\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Person\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"name\" value=\"java.lang.String\"/>\n                    <entry key=\"salary\" value=\"java.lang.Double\"/>\n                  </map>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n        \n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Organization\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Organization\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"name\" value=\"java.lang.String\"/>\n                  </map>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n      </list>\n    </property>\n  </bean>\n</beans>\n",
      "language": "xml",
      "name": "XML"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Note odbcConfiguration property.",
  "body": "Remember, you need explicitly enable ODBC on the node you going to connect to."
}
[/block]
As you can see, we have here two caches: `Person` and `Organization`, that contain types with matching names and some fields - `name` and `salary` for `Person` and `name` for `Organization`. Not much, but enough for the demonstration.
[block:callout]
{
  "type": "info",
  "title": "DiscoverySpi was not configured for this node.",
  "body": "You can read about how to configure it on [Cluster Configuration](doc:cluster-config) page."
}
[/block]
Of course you can still configure node from the Java code, as well as use `@QuerySqlField` annotations. Refer to [Cache Queries](doc:cache-queries) for details.
[block:api-header]
{
  "type": "basic",
  "title": "Connecting to the node"
}
[/block]
First you need to connect to configured Ignite node using ODBC. Nothing new here - you should properly specify connection string arguments. If some argument was not specified default value is used instead. Pay special attention to `Cache` attribute - you should specify name of any existing cache here. It is not really important which one thanks to [Cross-cache queries](doc:sql-queries#cross-cache-queries)