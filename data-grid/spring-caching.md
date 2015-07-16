Ignite is shipped with the `SpringCacheManager` - an implementation of [Spring Cache Abstraction](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html). It provides annotation-based way to enable caching for Java methods so that the result of a method execution is stored in the Ignite cache. If later the same method is called with the same set of parameters, the result will be retrieved from the cache instead of actually executing the method.
[block:callout]
{
  "type": "info",
  "title": "Spring Cache Abstraction documentation",
  "body": "For more information on how to use the Spring Cache Abstraction, including available annotations, refer to this documentation page: [http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/cache.html)"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "How to enable caching"
}
[/block]
Only two simple steps are required to plug in Ignite's cache into your Spring-based application:
* Start an Ignite node with proper configuration in embedded mode (i.e., in the same JVM where the application is running). It can already have predefined caches, but it's not required - caches will be created automatically on first access if needed.
* Configure `SpringCacheManager` as the cache manager in the Spring application context.

The embedded node can be started by `SpringCacheManager` itself. In this case you will need to provide a path to Ignite configuration XML file or `IgniteConfiguration` bean via `configurationPath` or `configuration` properties respectively (see examples below). Note that setting both is illegal and results in `IllegalArgumentException`.
[block:code]
{
  "codes": [
    {
      "code": "<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:cache=\"http://www.springframework.org/schema/cache\"\n       xsi:schemaLocation=\"\n         http://www.springframework.org/schema/beans\n         http://www.springframework.org/schema/beans/spring-beans.xsd\n         http://www.springframework.org/schema/cache\n         http://www.springframework.org/schema/cache/spring-cache.xsd\">\n    <-- Provide configuration file path. -->\n    <bean id=\"cacheManager\" class=\"org.apache.ignite.cache.spring.SpringCacheManager\">\n        <property name=\"configurationPath\" value=\"examples/config/spring-cache.xml\"/>\n    </bean>\n\n    <-- Enable annotation-driven caching. -->\n    <cache:annotation-driven/>\n</beans>",
      "language": "xml",
      "name": "Configuration path"
    },
    {
      "code": "<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:cache=\"http://www.springframework.org/schema/cache\"\n       xsi:schemaLocation=\"\n         http://www.springframework.org/schema/beans\n         http://www.springframework.org/schema/beans/spring-beans.xsd\n         http://www.springframework.org/schema/cache\n         http://www.springframework.org/schema/cache/spring-cache.xsd\">\n    <-- Provide configuration bean. -->\n    <bean id=\"cacheManager\" class=\"org.apache.ignite.cache.spring.SpringCacheManager\">\n        <property name=\"configuration\">\n            <bean class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n                 ...\n            </bean>\n        </property>\n    </bean>\n\n    <-- Enable annotation-driven caching. -->\n    <cache:annotation-driven/>\n</beans>",
      "language": "xml",
      "name": "Configuration bean"
    }
  ]
}
[/block]
It's possible that you already have an Ignite node running when the cache manager is initialized (e.g., it was started using `ServletContextListenerStartup`). In this case you should simply provide the grid name via `gridName` property. Note that if you don't set the grid name as well, the cache manager will try to use the default Ignite instance (the one with the `null` name). Here is the example:
[block:code]
{
  "codes": [
    {
      "code": "<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:cache=\"http://www.springframework.org/schema/cache\"\n       xsi:schemaLocation=\"\n         http://www.springframework.org/schema/beans\n         http://www.springframework.org/schema/beans/spring-beans.xsd\n         http://www.springframework.org/schema/cache\n         http://www.springframework.org/schema/cache/spring-cache.xsd\">\n    <-- Provide grid name. -->\n    <bean id=\"cacheManager\" class=\"org.apache.ignite.cache.spring.SpringCacheManager\">\n        <property name=\"gridName\" value=\"myGrid\"/>\n    </bean>\n\n    <-- Enable annotation-driven caching. -->\n    <cache:annotation-driven/>\n</beans>",
      "language": "xml",
      "name": "Using pre-started Ignite instance"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "success",
  "title": "Remote Nodes",
  "body": "Keep in mind that the node started inside your application is an entry point to the whole topology it connects to. You can start as many remote standalone nodes as you need using `bin/ignite.{sh|bat}` scripts provided in Ignite distribution, and all these nodes will participate in caching the data."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Dynamic Caches"
}
[/block]
While you can have all required caches predefined in Ignite configuration, it's not required. If Spring wants to use a cache that doesn't exist, the `SpringCacheManager` will automatically create it.

If otherwise not specified, a new cache will be created will all defaults. To customize it, you can provide a configuration template via `dynamicCacheConfiguration` property. For example, if you want to use `REPLICATED` caches instead of `PARTITIONED`, you should configure `SpringCacheManager` like this:
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"cacheManager\" class=\"org.apache.ignite.cache.spring.SpringCacheManager\">\n    ...\n  \n    <property name=\"dynamicCacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n            <property name=\"cacheMode\" value=\"REPLICATED\"/>\n        </bean>\n    </property>\n</bean>",
      "language": "xml",
      "name": "Dynamic cache configuration"
    }
  ]
}
[/block]
You can also utilize near caches on client side. To achieve this simply provide near cache configuration via `dynamicNearCacheConfiguration` property. By default near cache is not created. Here is the example:
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"cacheManager\" class=\"org.apache.ignite.cache.spring.SpringCacheManager\">\n    ...\n  \n    <property name=\"dynamicNearCacheConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.NearCacheConfiguration\">\n            <property name=\"nearStartSize\" value=\"1000\"/>\n        </bean>\n    </property>\n</bean>",
      "language": "xml",
      "name": "Dynamic near cache configuration"
    }
  ]
}
[/block]