* [Overview](#overview)
 * [Setting up](#setting-up)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Flume is a distributed, reliable, and available service for efficiently collecting, aggregating, and moving large amounts of log data." (https://github.com/apache/flume).

IgniteSink is a Flume sink that extracts Events from an associated Flume channel and injects into an Ignite cache. Flume 1.6.0 is supported.

IgniteSink and its dependencies have to be included in the agent's classpath, as described in the following subsection, before starting the Flume agent.
[block:api-header]
{
  "type": "basic",
  "title": "Setting up"
}
[/block]
1. Create a transformer by implementing EventTransformer interface.
2. Create 'ignite' directory inside plugins.d directory which is located in ${FLUME_HOME}. If the plugins.d directory is not there, create it.
3. Build it and copy to ${FLUME_HOME}/plugins.d/ignite-sink/lib.
4. Copy other Ignite-related jar files from Apache Ignite distribution to ${FLUME_HOME}/plugins.d/ignite-sink/libext to have them as shown below.
```
plugins.d/
`-- ignite
    |-- lib
    |   `-- ignite-flume-transformer-x.x.x.jar <-- your jar
    `-- libext
        |-- cache-api-1.0.0.jar
        |-- ignite-core-x.x.x.jar
        |-- ignite-flume-x.x.x.jar <-- IgniteSink
        |-- ignite-spring-x.x.x.jar
        |-- spring-aop-4.1.0.RELEASE.jar
        |-- spring-beans-4.1.0.RELEASE.jar
        |-- spring-context-4.1.0.RELEASE.jar
        |-- spring-core-4.1.0.RELEASE.jar
        `-- spring-expression-4.1.0.RELEASE.jar
```
5. In Flume configuration file, specify Ignite configuration XML file's location with cache properties (see *flume/src/test/resources/example-ignite.xml* for a basic example) with cache name specified for cache creation. Also specify the cache name (same as in Ignite configuration file), your EventTransformer's implementation class, and, optionally, batch size. All properties are shown in the table below (required properties are in bold).
[block:parameters]
{
  "data": {
    "0-0": "channel",
    "1-0": "type",
    "2-0": "igniteCfg",
    "3-0": "cacheName",
    "4-0": "eventTransformer",
    "5-0": "batchSize",
    "h-0": "Property name",
    "h-1": "Default",
    "h-2": "Description",
    "0-1": "-",
    "1-1": "-",
    "2-1": "-",
    "3-1": "-",
    "4-1": "-",
    "5-1": "100",
    "1-2": "The component type name. Needs to be org.apache.ignite.stream.flume.IgniteSink",
    "2-2": "Ignite configuration XML file",
    "3-2": "Cache name. Same as in igniteCfg",
    "4-2": "Your implementation of org.apache.ignite.stream.flume.EventTransformer",
    "5-2": "Number of events to be written per transaction"
  },
  "cols": 3,
  "rows": 6
}
[/block]
The sink configuration part of agent named *a1* can look like this:
```
a1.sinks.k1.type = org.apache.ignite.stream.flume.IgniteSink
a1.sinks.k1.igniteCfg = /some-path/ignite.xml
a1.sinks.k1.cacheName = testCache
a1.sinks.k1.eventTransformer = my.company.MyEventTransformer
a1.sinks.k1.batchSize = 100
```
After specifying your source and channel (see Flume's docs), you are ready to run a Flume agent.