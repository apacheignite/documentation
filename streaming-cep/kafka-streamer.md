Apache Ignite Kafka Streamer module provides streaming from Kafka to Ignite cache.

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