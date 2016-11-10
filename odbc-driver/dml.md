Starting from Ignite 1.8, DML is supported in SQL queries. This means DML can now be used in ODBC as well. On this you can find details.

With DML you can now insert, modify and delete records using SLQ alone, which means you can now manipulate data on Ignite writing virtually zero lines of Java code. Let me give you an example. Consider the following minimal node config:
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\n        http://www.springframework.org/schema/util\n        http://www.springframework.org/schema/util/spring-util.xsd\">\n  <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <!-- Enabling ODBC. -->\n    <property name=\"odbcConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.OdbcConfiguration\"></bean>\n    </property>\n\n    <!-- Configuring cache. -->\n    <property name=\"cacheConfiguration\">\n      <list>\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Person\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Person\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"firstName\" value=\"java.lang.String\"/>\n                    <entry key=\"lastName\" value=\"java.lang.String\"/>\n                    <entry key=\"resume\" value=\"java.lang.String\"/>\n                    <entry key=\"salary\" value=\"java.lang.Double\"/>\n                  </map>\n                </property>\n\n                <property name=\"indexes\">\n                  <list>\n                    <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                      <constructor-arg value=\"salary\"/>\n                    </bean>\n                  </list>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n      </list>\n    </property>\n  </bean>\n</beans>\n",
      "language": "xml",
      "name": "XML"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "",
  "body": "`DiscoverySpi` was not configured for this node. You can find details on how to configure it"
}
[/block]