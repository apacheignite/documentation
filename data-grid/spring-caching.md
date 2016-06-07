* [Overview](#overview)
* [How to Enable Caching](#how-to-enable-caching)
* [Dynamic Caches](##dynamic-caches)
* [Example](#example)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
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
      "code": "<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:cache=\"http://www.springframework.org/schema/cache\"\n       xsi:schemaLocation=\"\n         http://www.springframework.org/schema/beans\n         http://www.springframework.org/schema/beans/spring-beans.xsd\n         http://www.springframework.org/schema/cache\n         http://www.springframework.org/schema/cache/spring-cache.xsd\">\n    <!-- Provide configuration file path. -->\n    <bean id=\"cacheManager\" class=\"org.apache.ignite.cache.spring.SpringCacheManager\">\n        <property name=\"configurationPath\" value=\"examples/config/spring-cache.xml\"/>\n    </bean>\n\n    <!-- Enable annotation-driven caching. -->\n    <cache:annotation-driven/>\n</beans>",
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
      "code": "<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:cache=\"http://www.springframework.org/schema/cache\"\n       xsi:schemaLocation=\"\n         http://www.springframework.org/schema/beans\n         http://www.springframework.org/schema/beans/spring-beans.xsd\n         http://www.springframework.org/schema/cache\n         http://www.springframework.org/schema/cache/spring-cache.xsd\">\n    <!-- Provide grid name. -->\n    <bean id=\"cacheManager\" class=\"org.apache.ignite.cache.spring.SpringCacheManager\">\n        <property name=\"gridName\" value=\"myGrid\"/>\n    </bean>\n\n    <!-- Enable annotation-driven caching. -->\n    <cache:annotation-driven/>\n</beans>",
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

[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
Once you added `SpringCacheManager` to you Spring application context, you can enable caching for any Java method by simply attaching annotation to it.

Usually you would use caching for heavy operations, like database access. For example, let's assume you have a DAO class with `averageSalary(...)` method that calculates average salary of all employees in an organization. You can use `@Cacheable` annotation to enable caching for this method:
[block:code]
{
  "codes": [
    {
      "code": "private JdbcTemplate jdbc;\n\n@Cacheable(\"averageSalary\")\npublic long averageSalary(int organizationId) {\n    String sql =\n        \"SELECT AVG(e.salary) \" +\n        \"FROM Employee e \" +\n        \"WHERE e.organizationId = ?\";\n\n    return jdbc.queryForObject(sql, Long.class, organizationId);\n}",
      "language": "java"
    }
  ]
}
[/block]
When this method is called for the first time, `SpringCacheManager` will automatically create `averageSalary` cache. It will also lookup the pre-calculated average value in this cache and return it right away if it's there. If the average for this organization is not calculated yet, the method will be called and the result will be stored in cache. So next time you request average for this organization, you will not query the database.
[block:callout]
{
  "type": "info",
  "body": "Since `organizationId` is the only method parameter, it will be automatically used as a cache key.",
  "title": "Cache Key"
}
[/block]
If the salary of one of the employees is changed, you may want to remove the average value for the organization this employee belongs to, because otherwise `averageSalary(...)` method will return obsolete cached result. This can be achieved with `@CacheEvict` annotation attached to a method that updates employee's salary:
[block:code]
{
  "codes": [
    {
      "code": "private JdbcTemplate jdbc;\n\n@CacheEvict(value = \"averageSalary\", key = \"#e.organizationId\")\npublic void updateSalary(Employee e) {\n    String sql =\n        \"UPDATE Employee \" +\n        \"SET salary = ? \" +\n        \"WHERE id = ?\";\n  \n    jdbc.update(sql, e.getSalary(), e.getId());\n}",
      "language": "java"
    }
  ]
}
[/block]
After this method is called, average value for provided employee's organization will be evicted from `averageSalary` cache. This will force `averageSalary(...)` to recalculate the value next time it's called.
[block:callout]
{
  "type": "info",
  "title": "Spring Expression Language (SpEL)",
  "body": "Note that this method receives employee as a parameter, while average values are saved in cache by organization ID. To explicitly specify what is used as a cache key, we used `key` parameter of the annotation and [Spring Expression Language](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html).\n\n`#e.organizationId` expression means that we need to extract the value of `organizationId` property from `e` variable. Essentially, `getOrganizationId()` method will be called on provided employee object and the returned value will be used as the cache key."
}
[/block]