[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
[Apache Karaf](https://karaf.apache.org/) is a lightweight, powerful, and enterprise ready container powered by OSGi. It supports both the Eclipse Equinox and Apache Felix OSGi runtimes.
[block:callout]
{
  "type": "info",
  "title": "Apache Karaf 4.0.0 release line supported",
  "body": "Apache Ignite has been tested with the Apache Karaf 4.0.0 release line. It is possible that it works with older versions too, but it has not been tested explicitly."
}
[/block]
In order to facilitate the deployment of the different Ignite modules – along with their dependencies –, Apache Ignite offers a set of [Karaf features](https://karaf.apache.org/manual/latest/users-guide/provisioning.html) packaged in a feature repository. This makes it possible to quickly provision Ignite onto the OSGi environment by means of a single command in the Karaf shell.
[block:api-header]
{
  "type": "basic",
  "title": "Preparatory Steps"
}
[/block]
First off, Ignite uses the low-level Oracle/Sun JRE package `sun.nio.ch` (which is also available on OpenJDK).

Since this is a proprietary package (not part of the standard Java specs), Apache Karaf does not export it by default from the [System Bundle](http://wiki.osgi.org/wiki/System_Bundle) (bundle 0). You must instruct Karaf to export it by [modifying the `${KARAF_BASE}/etc/jre.properties` file](https://karaf.apache.org/manual/latest-2.2.x/users-guide/jre-tuning.html).

Locate the `jre-1.x` property for the version of the JRE you're on, and append the package at the end. For example:
[block:code]
{
  "codes": [
    {
      "code": "jre-1.8= \\\n javax.accessibility, \\\n javax.activation;version=\"1.1\", \\\n ...\n org.xml.sax.helpers, \\\n sun.nio.ch",
      "language": "text"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Installing Ignite Features Repository"
}
[/block]
Use the following command in the Apache Karaf shell to install the Ignite features repository, making sure your container has access to the Internet or to an alternate Maven repository containing the Ignite artifacts:
[block:code]
{
  "codes": [
    {
      "code": "karaf@root()> feature:repo-add mvn:org.apache.ignite/ignite-osgi-karaf/${ignite.version}/xml/features\nAdding feature url mvn:org.apache.ignite/ignite-osgi-karaf/${ignite.version}/xml/features\nkaraf@root()>",
      "language": "text",
      "name": "Adding the Ignite feature repository to Karaf"
    }
  ]
}
[/block]
Replace `${ignite.version}` with the version of Apache Ignite you'd like to install.

You should now be able to list all supported Ignite features:
[block:code]
{
  "codes": [
    {
      "code": "karaf@root()> feature:list | grep ignite\nignite-all                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: All\nignite-core                   | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Core\nignite-aop                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: AOP\nignite-aws                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: AWS\nignite-indexing               | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Indexing\nignite-hibernate              | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Hibernate\nignite-jcl                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: JCL\nignite-jms11                  | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: JMS 1.1\nignite-jta                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: JTA\nignite-kafka                  | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Kafka\n[...]\nkaraf@root()>",
      "language": "text",
      "name": "Listing Ignite features in Karaf"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Install Appropriate Ignite Features"
}
[/block]
The following features are special:

* `ignite-core`: the ignite-core module. It is required by all other features, so don't forget to install it.
* `ignite-all`: an aggregate feature that installs all of the other Ignite features.

All other features contain the corresponding Ignite module + its dependencies. You can install them in the following manner:
[block:code]
{
  "codes": [
    {
      "code": "karaf@root()> feature:install ignite-core\nkaraf@root()> feature:install ignite-kafka\nkaraf@root()> feature:install ignite-aop ignite-urideploy\nkaraf@root()>",
      "language": "text",
      "name": "Installing Ignite features in Karaf"
    }
  ]
}
[/block]
Some modules are OSGi fragments rather than bundles. When installing them, you may notice that either, or both, the Karaf shell and/or the `ignite-core` bundle restart.
[block:api-header]
{
  "type": "basic",
  "title": "ignite-log4j and Pax Logging"
}
[/block]

[block:callout]
{
  "type": "warning",
  "title": "Read this note carefully if using Pax Logging on Apache Karaf <= 4.0.3",
  "body": "When installing the `ignite-log4j` feature, the Karaf shell may show the following message:\n    \n    Error executing command: Resource has no uri\n    \nThis is not a harmful error and has been reported to the Karaf community at [KARAF-4129](https://issues.apache.org/jira/browse/KARAF-4129): Installing a feature with a fragment that attaches to pax-logging-api fails.\n\nApply the instructions below ignoring the error."
}
[/block]
Apache Karaf comes bundled with [Pax Logging](https://ops4j1.jira.com/wiki/display/paxlogging/Pax+Logging), a framework that collects and unifies all log output from other bundles (emitted via different frameworks such as slf4j, log4j, JULI, commons-logging, etc.) and processes it according to a canonical log4j configuration.

The `ignite-log4j` module required packages from log4j that Pax Logging doesn't export by default. We have developed an OSGi Fragment with symbolic name `ignite-osgi-paxlogging` that attaches to `ignite-core` and exports the missing packages.

The `ignite-log4j` feature already installs this fragment, but you will need to force its resolution by refreshing the bundle with symbolic name `org.ops4j.pax.logging.pax-logging-api`:
[block:code]
{
  "codes": [
    {
      "code": "karaf@root()> feature:install ignite-log4j\nkaraf@root()> refresh org.ops4j.pax.logging.pax-logging-api\nkaraf@root()>\n        __ __                  ____\n       / //_/____ __________ _/ __/\n      / ,<  / __ `/ ___/ __ `/ /_\n     / /| |/ /_/ / /  / /_/ / __/\n    /_/ |_|\\__,_/_/   \\__,_/_/\n\n  Apache Karaf (4.0.2)\n\nHit '<tab>' for a list of available commands\nand '[cmd] --help' for help on a specific command.\nHit '<ctrl-d>' or type 'system:shutdown' or 'logout' to shutdown Karaf.\n\nkaraf@root()> la | grep ignite-osgi-paxlogging\n75 | Resolved  |   8 | 1.5.0.SNAPSHOT                            | ignite-osgi-paxlogging, Hosts: 1\nkaraf@root()> ",
      "language": "text",
      "name": "Installing ignite-log4j in Karaf"
    }
  ]
}
[/block]