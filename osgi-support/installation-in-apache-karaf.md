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
  "title": "Installing the Ignite features repository"
}
[/block]
Use the following command in the Apache Karaf shell to install the Ignite features repository:
[block:code]
{
  "codes": [
    {
      "code": "karaf@root()> feature:repo-add mvn:org.apache.ignite/ignite-osgi-karaf/${ignite.version}/xml/features\nAdding feature url mvn:org.apache.ignite/ignite-osgi-karaf/${ignite.version}/xml/features\nkaraf@root()>",
      "language": "text"
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
      "code": "karaf@root()> feature:list | grep ignite\nignite-all                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: All\nignite-core                   | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Core\nignite-aop                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: AOP\nignite-aws                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: AWS\nignite-indexing               | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Indexing\nignite-hibernate              | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Hibernate\nignite-jcl                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: JCL\nignite-jms11                  | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: JMS 1.1\nignite-jta                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: JTA\nignite-kafka                  | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Kafka\nignite-mqtt                   | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: MQTT\nignite-log4j                  | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: log4j\nignite-rest-http              | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: REST HTTP\nignite-scalar-2.11            | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Scala 2.11\nignite-scalar-2.10            | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Scala 2.10\nignite-spring                 | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Spring Support\nignite-ssh                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: SSH\nignite-urideploy              | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: URI Deploy\nignite-web                    | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: Web\nignite-zookeeper              | 1.5.0.SNAPSHOT   |          | Uninstalled | ignite                   | Apache Ignite :: ZooKeeper\nkaraf@root()>",
      "language": "text"
    }
  ]
}
[/block]