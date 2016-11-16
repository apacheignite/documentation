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
  "type": "basic"
}
[/block]