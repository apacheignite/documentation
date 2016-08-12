* [Overview](#overview)
* [Streaming Data via Kafka Connect](#streaming-data-via-kafka-connect)
* [Streaming data with Ignite Kafka Streamer Module](#section-streaming-data-with-ignite-kafka-streamer-module)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite Kafka Streamer module provides streaming from Kafka to Ignite cache.
Either of the following two methods can be used to achieve such streaming:
- using Kafka Connect functionality with Ignite sink;
- importing Kafka Streamer module in your Maven project and instantiating KafkaStreamer for data streaming;
[block:api-header]
{
  "type": "basic",
  "title": "Streaming Data via Kafka Connect"
}
[/block]
IgniteSinkConnector will help you export data from Kafka to Ignite cache by polling data from Kafka topics and writing it to your specified cache.
Connector can be found in 'optional/ignite-kafka.' It and its dependencies have to be on the classpath of a Kafka running instance, as described in the following subsection.
*For more information on Kafka Connect, see [Kafka Documentation](http://kafka.apache.org/documentation.html#connect).*

### Setting up and Running

1. Put the following jar files on Kafka's classpath
[block:code]
{
  "codes": [
    {
      "code": "ignite-kafka-x.x.x.jar <-- with IgniteSinkConnector\nignite-core-x.x.x.jar\ncache-api-1.0.0.jar\nignite-spring-1.5.0-SNAPSHOT.jar\nspring-aop-4.1.0.RELEASE.jar\nspring-beans-4.1.0.RELEASE.jar\nspring-context-4.1.0.RELEASE.jar\nspring-core-4.1.0.RELEASE.jar\nspring-expression-4.1.0.RELEASE.jar\ncommons-logging-1.1.1.jar",
      "language": "text"
    }
  ]
}
[/block]
2. Prepare worker configurations, e.g.,
[block:code]
{
  "codes": [
    {
      "code": "bootstrap.servers=localhost:9092\n\nkey.converter=org.apache.kafka.connect.storage.StringConverter\nvalue.converter=org.apache.kafka.connect.storage.StringConverter\nkey.converter.schemas.enable=false\nvalue.converter.schemas.enable=false\n\ninternal.key.converter=org.apache.kafka.connect.storage.StringConverter\ninternal.value.converter=org.apache.kafka.connect.storage.StringConverter\ninternal.key.converter.schemas.enable=false\ninternal.value.converter.schemas.enable=false\n\noffset.storage.file.filename=/tmp/connect.offsets\noffset.flush.interval.ms=10000",
      "language": "text"
    }
  ]
}
[/block]
3. Prepare connector configurations, e.g.,
[block:code]
{
  "codes": [
    {
      "code": "# connector\nname=my-ignite-connector\nconnector.class=org.apache.ignite.stream.kafka.connect.IgniteSinkConnector\ntasks.max=2\ntopics=someTopic1,someTopic2\n\n# cache\ncacheName=myCache\ncacheAllowOverwrite=true\nigniteCfg=/some-path/ignite.xml",
      "language": "text"
    }
  ]
}
[/block]
where *cacheName* is the name of the cache you specify in '/some-path/ignite.xml' and the data from 'someTopic1,someTopic2' will be pulled and stored. *cacheAllowOverwrite* can be set to *true* if you want to enable overwriting existing values in the cache.
You can also set *cachePerNodeDataSize* and *cachePerNodeParOps* to adjust per-node buffer and the maximum number of parallel stream operations for a single node.

See *example-ignite.xml* in tests for a simple cache configuration file example.

4. Start connector, for instance, in a standalone mode as follows,
[block:code]
{
  "codes": [
    {
      "code": "bin/connect-standalone.sh myconfig/connect-standalone.properties myconfig/ignite-connector.properties",
      "language": "shell"
    }
  ]
}
[/block]
### Checking the Flow

To perform a very basic functionality check, you can do the following,

1. Start Zookeeper
[block:code]
{
  "codes": [
    {
      "code": "bin/zookeeper-server-start.sh config/zookeeper.properties",
      "language": "shell"
    }
  ]
}
[/block]
2. Start Kafka server
[block:code]
{
  "codes": [
    {
      "code": "bin/kafka-server-start.sh config/server.properties",
      "language": "shell"
    }
  ]
}
[/block]
3. Provide some data input to the Kafka server
[block:code]
{
  "codes": [
    {
      "code": "bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test --property parse.key=true --property key.separator=,\nk1,v1",
      "language": "shell"
    }
  ]
}
[/block]
4. Start the connector
[block:code]
{
  "codes": [
    {
      "code": "bin/connect-standalone.sh myconfig/connect-standalone.properties myconfig/ignite-connector.properties",
      "language": "shell"
    }
  ]
}
[/block]
5. Check the value is in the cache. For example, via REST API,
[block:code]
{
  "codes": [
    {
      "code": "http://node1:8080/ignite?cmd=size&cacheName=cache1",
      "language": "http"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Streaming data with Ignite Kafka Streamer Module"
}
[/block]
If you are using Maven to manage dependencies of your project, first of all you will have to add Kafka Streamer module dependency like this (replace '${ignite.version}' with actual Ignite version you are interested in):
[block:code]
{
  "codes": [
    {
      "code": "<project xmlns=\"http://maven.apache.org/POM/4.0.0\"\n    xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n    xsi:schemaLocation=\"http://maven.apache.org/POM/4.0.0\n                        http://maven.apache.org/xsd/maven-4.0.0.xsd\">\n    ...\n    <dependencies>\n        ...\n        <dependency>\n            <groupId>org.apache.ignite</groupId>\n            <artifactId>ignite-kafka</artifactId>\n            <version>${ignite.version}</version>\n        </dependency>\n        ...\n    </dependencies>\n    ...\n</project>",
      "language": "xml"
    }
  ]
}
[/block]
Having a cache with *String* keys and *String* values, the streamer can be started as follows
[block:code]
{
  "codes": [
    {
      "code": "KafkaStreamer<String, String, String> kafkaStreamer = new KafkaStreamer<>();\n\ntry (IgniteDataStreamer<String, String> stmr = ignite.dataStreamer(null)) {\n    // allow overwriting cache data\n    stmr.allowOverwrite(true);\n    \n    kafkaStreamer.setIgnite(ignite);\n    kafkaStreamer.setStreamer(stmr);\n    \n    // set the topic\n    kafkaStreamer.setTopic(someKafkaTopic);\n\n    // set the number of threads to process Kafka streams\n    kafkaStreamer.setThreads(4);\n    \n    // set Kafka consumer configurations\n    kafkaStreamer.setConsumerConfig(kafkaConsumerConfig);\n    \n    // set decoders\n    kafkaStreamer.setKeyDecoder(strDecoder);\n    kafkaStreamer.setValueDecoder(strDecoder);\n    \n    kafkaStreamer.start();\n}\nfinally {\n    kafkaStreamer.stop();\n}",
      "language": "java"
    }
  ]
}
[/block]
For the detailed information on Kafka consumer properties, refer http://kafka.apache.org/documentation.html