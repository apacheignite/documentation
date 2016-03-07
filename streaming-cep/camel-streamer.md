This documentation page focuses on the **[Apache Camel](http://camel.apache.org) Streamer**, which can also be thought of as a _Universal Streamer_ because it allows you to consume from any technology or protocol supported by Camel into an Ignite Cache. 
[block:callout]
{
  "type": "info",
  "title": "What is Apache Camel?",
  "body": "If you don't know what **[Apache Camel](http://camel.apache.org)** is, check out the section at the bottom of the page for a quick introduction."
}
[/block]
With this streamer, you can ingest entries straight into an Ignite cache based on:

* Calls received on a Web Service (SOAP or REST), by extracting the body or headers.
* Listening on a TCP or UDP channel for messages.
* The content of files received via FTP or written to the local filesystem.
* Email messages received via POP3 or IMAP.
* A MongoDB tailable cursor.
* An AWS SQS queue.
* And many others.

This streamer supports two modes of ingestion: **direct ingestion** and **mediated ingestion**.
[block:callout]
{
  "type": "warning",
  "body": "There is also a [`camel-ignite` component](https://camel.apache.org/ignite.html), if what you are looking is to interact with Ignite Caches, Compute, Events, Messaging, etc. from within a Camel route.",
  "title": "An Ignite Camel component"
}
[/block]

[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/QZPsu44JR6C5Mb6MkwVi",
        "Screen-Shot-2016-01-28-at-18-04-41.png",
        "896",
        "508",
        "#b6282a",
        ""
      ],
      "caption": "Architectural view of the Ignite Camel Streamer."
    }
  ]
}
[/block]
Read on for more details.
[block:api-header]
{
  "type": "basic",
  "title": "Direct Ingestion"
}
[/block]
Direct Ingestion allows you to consume from any Camel endpoint straight into Ignite, with the help of a Tuple Extractor. We call this **direct ingestion**. 

Here is a code sample:
[block:code]
{
  "codes": [
    {
      "code": "// Start Apache Ignite.\nIgnite ignite = Ignition.start();\n\n// Create an streamer pipe which ingests into the 'mycache' cache.\nIgniteDataStreamer<String, String> pipe = ignite.dataStreamer(\"mycache\");\n\n// Create a Camel streamer and connect it.\nCamelStreamer<String, String> streamer = new CamelStreamer<>();  \nstreamer.setIgnite(ignite);  \nstreamer.setStreamer(pipe);\n\n// This endpoint starts a Jetty server and consumes from all network interfaces on port 8080 and context path /ignite.\nstreamer.setEndpointUri(\"jetty:http://0.0.0.0:8080/ignite?httpMethodRestrict=POST\");\n\n// This is the tuple extractor. We'll assume each message contains only one tuple.\n// If your message contains multiple tuples, use a StreamMultipleTupleExtractor.\n// The Tuple Extractor receives the Camel Exchange and returns a Map.Entry<?,?> with the key and value.\nstreamer.setSingleTupleExtractor(new StreamSingleTupleExtractor<Exchange, String, String>() {  \n    @Override public Map.Entry<String, String> extract(Exchange exchange) {\n        String stationId = exchange.getIn().getHeader(\"X-StationId\", String.class);\n        String temperature = exchange.getIn().getBody(String.class);\n        return new GridMapEntry<>(stationId, temperature);\n    }\n});\n\n// Start the streamer.\nstreamer.start();  ",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Mediated Ingestion"
}
[/block]
For more sophisticated scenarios, you can also create a Camel route that performs complex processing on incoming messages, e.g. transformations, validations, splitting, aggregating, idempotency, resequencing, enrichment, etc. and **ingest only the result into the Ignite cache**. 

We call this **mediated ingestion**.
[block:code]
{
  "codes": [
    {
      "code": "// Create a CamelContext with a custom route that will:\n//  (1) consume from our Jetty endpoint.\n//  (2) transform incoming JSON into a Java object with Jackson.\n//  (3) uses JSR 303 Bean Validation to validate the object.\n//  (4) dispatches to the direct:ignite.ingest endpoint, where the streamer is consuming from.\nCamelContext context = new DefaultCamelContext();  \ncontext.addRoutes(new RouteBuilder() {  \n    @Override\n    public void configure() throws Exception {\n        from(\"jetty:http://0.0.0.0:8080/ignite?httpMethodRestrict=POST\")\n            .unmarshal().json(JsonLibrary.Jackson)\n            .to(\"bean-validator:validate\")\n            .to(\"direct:ignite.ingest\");\n    }\n});\n\n// Remember our Streamer is now consuming from the Direct endpoint above.\nstreamer.setEndpointUri(\"direct:ignite.ingest\");",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Setting a response"
}
[/block]
By default, the response sent back to the caller (if it is a synchronous endpoint) is simply an echo of the original request. If you want to customise the response, set a Camel `Processor` as a `responseProcessor`:
[block:code]
{
  "codes": [
    {
      "code": "streamer.setResponseProcessor(new Processor() {  \n    @Override public void process(Exchange exchange) throws Exception {\n        exchange.getOut().setHeader(Exchange.HTTP_RESPONSE_CODE, 200);\n        exchange.getOut().setBody(\"OK\");\n    }\n});",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Maven Dependency"
}
[/block]
To make use of the `ignite-camel` streamer, you need to add the following dependency:
[block:code]
{
  "codes": [
    {
      "code": "<dependency>\n\t<groupId>org.apache.ignite</groupId>\n\t<artifactId>ignite-camel</artifactId>\n\t<version>${ignite.version}</version>\n</dependency>",
      "language": "xml"
    }
  ]
}
[/block]
It will also drag in `camel-core` as a transitive dependency.
[block:callout]
{
  "type": "danger",
  "body": "Make sure to also add the dependencies to the Camel components that you'll be using in the streamer.",
  "title": "Don't forget to add the Camel component dependencies!"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "About Apache Camel"
}
[/block]
[Apache Camel](http://camel.apache.org) is an enterprise integration framework that revolves around the idea of the well-known [Enterprise Integration Patterns](http://www.enterpriseintegrationpatterns.com/) popularised by Gregor Hohpe and Bobby Woolf – such as _channels, pipes, filters, splitters, aggregators, routers, resequencers, etc._ – which you piece with one another like a Lego puzzle to create integration routes that connect systems together.

To date, there are over 200 components for Camel, many of which are adapters for different technologies like **JMS, SOAP, HTTP, Files, FTP, POP3, SMTP, SSH**; including cloud services like **Amazon Web Services, Google Compute Engine, Salesforce**; social networks like **Twitter, Facebook**; and even new generation databases like **MongoDB, Cassandra**; and data processing technologies like **Hadoop (HDFS, HBase) and Spark**.

Camel runs in a variety of environments, also supported by Ignite: standalone Java, OSGi, Servlet containers, Spring Boot, JEE application servers, etc. and it's fully modular, so you only deploy the components you'll actually be using and nothing else.

Check out [What is Camel?](https://camel.apache.org/what-is-camel.html) for more information.