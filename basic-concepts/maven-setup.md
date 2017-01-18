* [Overview](#overview)
* [Common Dependencies](#common-dependencies)
* [Importing Individual Modules A La Carte](#importing-individual-modules-a-la-carte)
* [LGPL Dependencies](#lgpl-dependencies)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
If you are using Maven to manage dependencies of your project, you can import individual Ignite modules a la carte.
[block:callout]
{
  "type": "info",
  "body": "In the examples below, please replace `${ignite.version}` with actual Apache Ignite version you are interested in."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Common Dependencies"
}
[/block]
Ignite data fabric comes with one mandatory dependency on `ignite-core.jar`. 
[block:code]
{
  "codes": [
    {
      "code": "<dependency>\n    <groupId>org.apache.ignite</groupId>\n    <artifactId>ignite-core</artifactId>\n    <version>${ignite.version}</version>\n</dependency>",
      "language": "xml"
    }
  ]
}
[/block]
However, in many cases, you may wish to have more dependencies, for example, if you want to use Spring configuration or SQL queries.

Here are the most commonly used optional modules:
  * ignite-indexing (optional, add if you need SQL indexing)
  * ignite-spring (optional, add if you plan to use Spring configuration) 
[block:code]
{
  "codes": [
    {
      "code": "<dependency>\n    <groupId>org.apache.ignite</groupId>\n    <artifactId>ignite-core</artifactId>\n    <version>${ignite.version}</version>\n</dependency>\n<dependency>\n    <groupId>org.apache.ignite</groupId>\n    <artifactId>ignite-spring</artifactId>\n    <version>${ignite.version}</version>\n</dependency>\n<dependency>\n    <groupId>org.apache.ignite</groupId>\n    <artifactId>ignite-indexing</artifactId>\n    <version>${ignite.version}</version>\n</dependency>",
      "language": "xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Importing Individual Modules A La Carte"
}
[/block]
You can import Ignite modules a la carte, one by one. The only required module is `ignite-core`, all others are optional. All optional modules can be imported just like the core module, but with different artifact IDs.

The following modules are available:
  * `ignite-spring` (for Spring-based configuration support)
  * `ignite-indexing` (for SQL querying and indexing)
  * `ignite-geospatial` (for geospatial indexing)
  * `ignite-hibernate` (for Hibernate integration)
  * `ignite-web` (for Web Sessions Clustering)
  * `ignite-schedule` (for Cron-based task scheduling)
  * `ignite-logj4` (for Log4j logging)
  * `ignite-jcl` (for Apache Commons logging)
  * `ignite-jta` (for XA integration)
  * `ignite-hadoop2-integration` (Integration with HDFS 2.0)
  * `ignite-rest-http` (for HTTP REST messages)
  * `ignite-scalar` (for Ignite Scala API)
  * `ignite-slf4j` (for SLF4J logging)
  * `ignite-ssh` (for starting grid nodes on remote machines)
  * `ignite-urideploy` (for URI-based deployment)
  * `ignite-aws` (for seamless cluster discovery on AWS S3)
  * `ignite-aop` (for AOP-based grid-enabling)
  * `ignite-visor-console`  (open source command line management and monitoring tool)
[block:callout]
{
  "type": "warning",
  "title": "Artifact versions",
  "body": "Note that when importing several Ignite module, the version has to the same for all of them. E.g., if you're using `ignite-core` 1.7, all other modules must be imported with the version 1.7 as well."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "LGPL Dependencies"
}
[/block]
The following Ignite modules have LGPL dependencies and therefore can't be deployed on Maven Central repository:
* `ignite-hibernate`
* `ignite-geospatial`
* `ignite-schedule`

To use these modules, you will need to build them from sources manually and add them to your project. For example, to install `ignite-hibernate` into your local repository, run this command in the Ignite source package:
[block:code]
{
  "codes": [
    {
      "code": "mvn clean install -DskipTests -Plgpl -pl modules/hibernate -am",
      "language": "shell",
      "name": ""
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "3rd party repositories",
  "body": "GridGain provides his own [Maven repository](http://www.gridgainsystems.com/nexus/content/repositories/external) containing Apache Ignite LGPL artifacts such as `ignite-hibernate`.\nPlease note that artifacts located at GridGain Maven repository provided for convenience and are NOT official Apache Ignite artifacts."
}
[/block]