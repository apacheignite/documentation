[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
In addition to conventional ANSI-99 SQL queries that are supported by Ignite and executed over primitive data types or objects of specific/custom types, the platform also allows querying and indexing geometry data types such as points, lines and polygons considering the spatial relationship between these geometries.

Spatial queries' capabilities, as well as available functions and operands, are defined by [Simple Features Specification for SQL](http://www.opengeospatial.org/docs/is/). The specification is fully implemented by [JTS Topology Suite](http://tsusiatsoftware.net/jts/main.html) which is used by Apache Ignite along with [H2](http://www.h2database.com/html/advanced.html?highlight=jts&search=its#firstFound) in order to build the unique geospatial component that works in a distributed and fault-tolerant fashion.
[block:api-header]
{
  "type": "basic",
  "title": "Including Ignite Geospatial Library"
}
[/block]
Ignite geospatial library (`ignite-geospatial`) depends on [JTS](http://tsusiatsoftware.net/jts/main.html) that is available under LGPL license that is not aligned with Apache license and prevents from including `ignite-geospatial` into Apache Ignite binary deliveries.

Due to this reason, a binary version of `ignite-geospatial` library is hosted in the Maven repository shown below:
 
[block:code]
{
  "codes": [
    {
      "code": "<repositories>\n\t<repository>\n  \t<id>GridGain External Repository</id>             \t<url>http://www.gridgainsystems.com/nexus/content/repositories/external</url>\n\t</repository>\n</repositories>",
      "language": "xml"
    }
  ]
}
[/block]
Add this repository and the Maven dependency below to make sure that the geospatial library is included into your application:
[block:code]
{
  "codes": [
    {
      "code": "<dependency>\n\t<groupId>org.apache.ignite</groupId>\n  <artifactId>ignite-geospatial</artifactId>\n  <version>${ignite.version}</version>\n</dependency>",
      "language": "xml"
    }
  ]
}
[/block]
Alternatively, you can download Apache Ignite in sources and built the library on your own.
[block:api-header]
{
  "type": "basic",
  "title": "Executing Geospatial Queries"
}
[/block]
The geospatial module works only for the objects of `com.vividsolutions.jts` type. 

To configure indexes of geometry types you need to use the same approaches that exist and used for non-geospatial types.  First, the indexes can be defined with the help of `org.apache.ignite.cache.QueryEntity` which is convenient for Spring XML based configurations. Second, you can achieve the same outcome by annotating indexes with `@QuerySqlField` annotations which will be converted `QueryEntities` internally.
[block:code]
{
  "codes": [
    {
      "code": "/**\n * Point with indexed coordinates.\n */\nprivate static class Point {\n    /** Coordinates. */\n    @QuerySqlField(index = true)\n    private Geometry coords;\n\n    /**\n     * @param coords Coordinates.\n     */\n    private Point(Geometry coords) {\n        this.coords = coords;\n    }\n}",
      "language": "java"
    },
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"name\" value=\"mycache\"/>\n    <!-- Configure query entities -->\n    <property name=\"queryEntities\">\n        <list>\n            <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Integer\"/>\n                <property name=\"valueType\" value=\"org.apache.ignite.examples.Point\"/>\n\n                <property name=\"fields\">\n                    <map>\n                        <entry key=\"coords\" value=\"com.vividsolutions.jts.geom.Geometry\"/>\n                    </map>\n                </property>\n\n                <property name=\"indexes\">\n                    <list>\n                        <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                            <constructor-arg value=\"coords\"/>\n                        </bean>\n                    </list>\n                </property>\n            </bean>\n        </list>\n    </property>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]