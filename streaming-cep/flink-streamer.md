Apache Ignite Flink Sink module is a streaming connector to inject Flink data into Ignite cache. The sink emits its input data to Ignite cache. When creating a sink, a Ignite cache name and Ignite grid configuration file have to be provided. 

Starting data transfer to Ignite cache can be done with the following steps.

1. Import Ignite Flink Sink Module in Maven Project
If you are using Maven to manage dependencies of your project, you can add Flink module
dependency like this (replace '${ignite.version}' with actual Ignite version you are
interested in):
[block:code]
{
  "codes": [
    {
      "code": "<project xmlns=\"http://maven.apache.org/POM/4.0.0\"\n    xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n    xsi:schemaLocation=\"http://maven.apache.org/POM/4.0.0\n                        http://maven.apache.org/xsd/maven-4.0.0.xsd\">\n    ...\n    <dependencies>\n        ...\n        <dependency>\n            <groupId>org.apache.ignite</groupId>\n            <artifactId>ignite-flink</artifactId>\n            <version>${ignite.version}</version>\n        </dependency>\n        ...\n    </dependencies>\n    ...\n</project>",
      "language": "xml"
    }
  ]
}
[/block]
2. Create an Ignite configuration file and make sure it is accessible from the sink.
3. Make sure your data input to the sink is specified and start the sink.
[block:code]
{
  "codes": [
    {
      "code": "IgniteSink igniteSink = new IgniteSink(\"myCache\", \"ignite.xml\");\n\nigniteSink.setAllowOverwrite(true);\nigniteSink.setAutoFlushFrequency(10);\nigniteSink.start();\n\nDataStream<Map> stream = ...;\n\n// Sink data into the grid.\nstream.addSink(igniteSink);\ntry {\n    env.execute();\n} catch (Exception e){\n    // Exception handling.\n}\nfinally {\n    igniteSink.stop();\n}",
      "language": "java"
    }
  ]
}
[/block]
Refer to the Javadocs of the ignite-flink module for more info on the available options.