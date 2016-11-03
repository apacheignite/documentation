[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
PHP provides a lightweight, consistent interface for accessing databases named PHP Data Objects   - PDO. This extension goes with several database-specific PDO drivers. One of them is [PDO_ODBC](http://php.net/manual/en/ref.pdo-odbc.php), which allows connecting to any database that provides its own ODBC driver.

With the usage of Apache Ignite's ODBC driver it's possible to connect to an Apache Ignite cluster from a PHP application accessing and modifying data that is stored there. This page provides instructions on how to bring this to life.
[block:api-header]
{
  "type": "basic",
  "title": "Setting Up ODBC Driver"
}
[/block]
Apache Ignite conforms to ODBC protocol and has its own ODBC driver that is released along with other functionality. This is the driver that will be used by PHP PDO framework going forward in order to connect to an Apache Ignite cluster.

Refer to Apache Ignite [ODBC documentation](doc:odbc-driver) configuring it and installing on a target system. Once the driver is installed and works fine move to the section below of this guide.
[block:callout]
{
  "type": "warning",
  "title": "",
  "body": "Use ODBC driver that is available under Apache Ignite 1.8 and later versions. The prior versions of the driver and Apache Ignite don't support PHP PDO framework."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Installing and Configuring PHP PDO"
}
[/block]
To install PHP, PDO and PDO_ODBC driver refer to the generic PHP resources:
* [Download](http://php.net/downloads.php) and install the desired PHP version. Note, that PDO driver is enabled by default in PHP as of PHP 5.1.0.
* [Configure](http://php.net/manual/en/book.pdo.php) PHP PDO framework.
* [Enable](http://php.net/manual/en/ref.pdo-odbc.php) PDO_ODBC driver.
* If necessary, [configure](http://php.net/manual/en/ref.pdo-odbc.php#ref.pdo-odbc.installation) PDO_ODBC driver for your system. In most cases, however, simple installation of the PDO_ODBC driver is going to be enough.
[block:api-header]
{
  "type": "basic",
  "title": "Starting Ignite Cluster"
}
[/block]
After PHP PDO is installed and ready to be used let's start an Ignite cluster with an exemplary configuration and connect to the cluster from a PHP application updating and querying cluster's data.

First of all, you should enable ODBC processor on the node, you are going to connect to. To do so you need to specify

[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n                           http://www.springframework.org/schema/beans\n                           http://www.springframework.org/schema/beans/spring-beans.xsd\n                           http://www.springframework.org/schema/util\n                           http://www.springframework.org/schema/util/spring-util.xsd\">\n  <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <!-- Enabling ODBC. -->\n    <property name=\"odbcConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.OdbcConfiguration\"></bean>\n    </property>\n\n    <!-- Configuring cache. -->\n    <property name=\"cacheConfiguration\">\n      <list>\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Person\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Person\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"orgId\" value=\"java.lang.Long\"/>\n                    <entry key=\"firstName\" value=\"java.lang.String\"/>\n                    <entry key=\"lastName\" value=\"java.lang.String\"/>\n                    <entry key=\"resume\" value=\"java.lang.String\"/>\n                    <entry key=\"salary\" value=\"java.lang.Double\"/>\n                  </map>\n                </property>\n\n                <property name=\"indexes\">\n                  <list>\n                    <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                      <constructor-arg value=\"salary\"/>\n                    </bean>\n                  </list>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n      </list>\n    </property>\n  </bean>\n</beans>\n",
      "language": "xml",
      "name": "XML"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Connecting From PHP to Ignite Cluster"
}
[/block]
TBD: more details to be added

You also are going to need properly configured DSN for Ignite. In the example below we assume that your DSN's name is "Apache Ignite DSN".
[block:callout]
{
  "type": "info",
  "title": "Note that you can't connect with PDO ODBC without properly configured DSN."
}
[/block]
Once you have all these things, you can finally write some code using PDO to connect to Apache Ignite node.
[block:code]
{
  "codes": [
    {
      "code": "<?php\ntry {\n  $dbh = new PDO('odbc:Apache Ignite DSN');\n  \n} catch (PDOException $e) {\n  print \"Error!: \" . $e->getMessage() . \"\\n\";\n  die();\n}\n?>",
      "language": "php"
    }
  ]
}
[/block]
All done. Now you can use your PDO connection with Apache Ignite as any other PDO connection.