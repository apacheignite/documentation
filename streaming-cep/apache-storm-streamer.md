Apache Ignite Storm Streamer module provides streaming via Storm (http://storm.apache.org/) to Ignite cache.

Starting data transfer to Ignite cache can be done with the following steps.

1. Import Ignite Storm Streamer Module In Maven Project.
If you are using Maven to manage dependencies of your project, you can add Storm module dependency like this (replace '${ignite.version}' with actual Ignite version you are interested in):
[block:code]
{
  "codes": [
    {
      "code": "<project xmlns=\"http://maven.apache.org/POM/4.0.0\"\n    xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n    xsi:schemaLocation=\"http://maven.apache.org/POM/4.0.0\n                        http://maven.apache.org/xsd/maven-4.0.0.xsd\">\n    ...\n    <dependencies>\n        ...\n        <dependency>\n            <groupId>org.apache.ignite</groupId>\n            <artifactId>ignite-storm</artifactId>\n            <version>${ignite.version}</version>\n        </dependency>\n        ...\n    </dependencies>\n    ...\n</project>",
      "language": "xml"
    }
  ]
}
[/block]
2. Create an Ignite configuration file (see example-ignite.xml in modules/storm/src/test/resources/example-ignite.xml) and make sure it is accessible from the streamer.

3. Make sure your key-value data input to the streamer is specified with the field named "ignite" (or a different one you configure with StormStreamer.setIgniteTupleField(...)).
See TestStormSpout.declareOutputFields(...) for an example.

4. Create a topology with the streamer, make a jar file with all dependencies and run like the following.
[block:code]
{
  "codes": [
    {
      "code": "storm jar ignite-storm-streaming-jar-with-dependencies.jar my.company.ignite.MyStormTopology",
      "language": "shell"
    }
  ]
}
[/block]