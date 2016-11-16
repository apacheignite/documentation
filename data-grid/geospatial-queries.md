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