From version 1.5 Ignite can be used as a 2nd level cache of MyBatis for performance.

If you are an Apache Maven user, simply add the following dependency to the pom.xml.
[block:code]
{
  "codes": [
    {
      "code": "<dependencies>\n  ...\n  <dependency>\n    <groupId>org.mybatis.caches</groupId>\n    <artifactId>mybatis-ignite</artifactId>\n    <version>1.0.0</version>\n  </dependency>\n  ...\n</dependencies>",
      "language": "xml"
    }
  ]
}
[/block]
Or you can also download the [zip bundle](https://github.com/mybatis/ignite-cache/releases), decompress it and add the jars in the classpath.

Then, just specify it in the mapper XML as follows
[block:code]
{
  "codes": [
    {
      "code": "<mapper namespace=\"org.acme.FooMapper\">\n  <cache type=\"org.mybatis.caches.ignite.IgniteCacheAdapter\" />\n</mapper>",
      "language": "xml"
    }
  ]
}
[/block]
and configure your Ignite cache in *config/default-config.xml*.  (Simple reference configurations are available on [github](https://github.com/mybatis/ignite-cache/tree/master/config))
[block:callout]
{
  "type": "warning",
  "body": "As of current implementation, EvictionPolicy, CacheLoaderFactory, CacheWriterFactory cannot be enabled in *config/default-config.xml*"
}
[/block]
For more details on MyBatis caching feature, refer [MyBatis docs](http://www.mybatis.org/mybatis-3/sqlmap-xml.html#cache).