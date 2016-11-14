[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
PHP provides a lightweight, consistent interface for accessing databases named PHP Data Objects   - PDO. This extension goes with several database-specific PDO drivers. One of them is [PDO_ODBC](http://php.net/manual/en/ref.pdo-odbc.php), which allows connecting to any database that provides its own ODBC driver implementation.

With the usage of Apache Ignite's ODBC driver, it's possible to connect to an Apache Ignite cluster from a PHP application accessing and modifying data that is stored there. This page provides instructions on how to bring this to life.
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
 * On Windows it may be needed to uncomment `extension=php_pdo_odbc.dll` line in the php.ini and make sure that `extension_dir` points to a directory which contains `php_pdo_odbc.dll`. Moreover, this directory has to be added to `PATH` environment variable.
 * On Unix based systems most often it's simply required to install a special PHP_ODBC package. For instance, `php5-odbc` package has to be installed on Ubuntu 14.04.
* If necessary, [configure](http://php.net/manual/en/ref.pdo-odbc.php#ref.pdo-odbc.installation) and build PDO_ODBC driver for a specific system that does not fall under a general case. In most cases, however, simple installation of both PHP and PDO_ODBC driver is going to be enough.
[block:api-header]
{
  "type": "basic",
  "title": "Starting Ignite Cluster"
}
[/block]
After PHP PDO is installed and ready to be used let's start an Ignite cluster with an exemplary configuration and connect to the cluster from a PHP application updating and querying cluster's data.

First of all, ODBC processor has to be enabled cluster wide. To do so, `odbcConfiguration` property has to be added to `IgniteConfiguration` of every cluster node.

Next, list configurations for all the caches related to specific data models inside of `IgniteConfiguration`. Since we're going to execute SQL queries from PHP PDO side over the cluster, every cache configuration needs to contain a definition for `QueryEntity`. Please refer to [SQL Queries](doc:sql-queries) documentation to learn more about query entities and SQL queries in Ignite.

Use the configuration template below and start an Ignite cluster using one of the methods described in [Getting Started](doc:getting-started#start-from-command-line).  
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\n        http://www.springframework.org/schema/util\n        http://www.springframework.org/schema/util/spring-util.xsd\">\n  <bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <!-- Enabling ODBC. -->\n    <property name=\"odbcConfiguration\">\n      <bean class=\"org.apache.ignite.configuration.OdbcConfiguration\"></bean>\n    </property>\n\n    <!-- Configuring cache. -->\n    <property name=\"cacheConfiguration\">\n      <list>\n        <bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n          <property name=\"name\" value=\"Person\"/>\n          <property name=\"cacheMode\" value=\"PARTITIONED\"/>\n          <property name=\"atomicityMode\" value=\"TRANSACTIONAL\"/>\n          <property name=\"writeSynchronizationMode\" value=\"FULL_SYNC\"/>\n\n          <property name=\"queryEntities\">\n            <list>\n              <bean class=\"org.apache.ignite.cache.QueryEntity\">\n                <property name=\"keyType\" value=\"java.lang.Long\"/>\n                <property name=\"valueType\" value=\"Person\"/>\n\n                <property name=\"fields\">\n                  <map>\n                    <entry key=\"firstName\" value=\"java.lang.String\"/>\n                    <entry key=\"lastName\" value=\"java.lang.String\"/>\n                    <entry key=\"resume\" value=\"java.lang.String\"/>\n                    <entry key=\"salary\" value=\"java.lang.Double\"/>\n                  </map>\n                </property>\n\n                <property name=\"indexes\">\n                  <list>\n                    <bean class=\"org.apache.ignite.cache.QueryIndex\">\n                      <constructor-arg value=\"salary\"/>\n                    </bean>\n                  </list>\n                </property>\n              </bean>\n            </list>\n          </property>\n        </bean>\n      </list>\n    </property>\n  </bean>\n</beans>\n",
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
To connect to Ignite from PHP PDO side the DSN has to be properly configured for Ignite. See [Configuring DSN](connection-string-and-dsn#configuring-dsn) section for details. In the example below it's assumed that DSN's name is "Apache Ignite DSN".
[block:callout]
{
  "type": "info",
  "title": "Note that you can't connect using PDO ODBC without properly configured DSN."
}
[/block]
At the end, once everything is configured and can be inter-connected it's time to connect to the Apache Ignite cluster from a PHP PDO application and execute a number of queries like the ones shown below.
[block:code]
{
  "codes": [
    {
      "code": "<?php\ntry {\n  // Connecting to Ignite using pre-configured DSN.\n  $dbh = new PDO('odbc:Apache Ignite DSN');\n\n  // Preparing query.\n  $dbs = $dbh->prepare('INSERT INTO Person (_key, firstName, lastName, resume, salary) VALUES (?, ?, ?, ?, ?)');\n\n  // Declaringing parameters.\n  $key = 777;\n  $firstName = \"James\";\n  $lastName = \"Bond\";\n  $resume = \"Secret Service agent\";\n  $salary = 7777777;\n\n  // Binding parameters.\n  $dbs->bindParam(1, $key);\n  $dbs->bindParam(2, $firstName);\n  $dbs->bindParam(3, $lastName);\n  $dbs->bindParam(4, $resume);\n  $dbs->bindParam(5, $salary);\n  \n  // Executing query.\n  $dbs->execute();\n  \n  // Updating parameters.\n  $key = 42;\n  $firstName = \"Arthur\";\n  $lastName = \"Dent\";\n  $resume = \"No info\";\n  $salary = 0.0;\n  \n  // Executing query again with updated parameters.\n  $dbs->execute();\n\n} catch (PDOException $e) {\n  print \"Error!: \" . $e->getMessage() . \"\\n\";\n  die();\n}\n?>",
      "language": "php",
      "name": "Insert"
    },
    {
      "code": "<?php\ntry {\n  // Connecting to Ignite using pre-configured DSN.\n  $dbh = new PDO('odbc:Apache Ignite DSN');\n\n  // Performing query.\n  $dbh->query('UPDATE Person SET salary = 2000.0 WHERE salary < 2000.0');\n\n} catch (PDOException $e) {\n  print \"Error!: \" . $e->getMessage() . \"\\n\";\n  die();\n}\n?>",
      "language": "php",
      "name": "Update"
    },
    {
      "code": "<?php\ntry {\n  // Connecting to Ignite using pre-configured DSN.\n  $dbh = new PDO('odbc:Apache Ignite DSN');\n\n  // Performing query and getting result.\n  $res = $dbh->query('SELECT firstName, lastName, resume, salary from Person WHERE salary < 2000.0');\n\n  // Printing results.\n  foreach($res as $row) {\n    print_r($row);\n  }\n\n} catch (PDOException $e) {\n  print \"Error!: \" . $e->getMessage() . \"\\n\";\n  die();\n}\n?>",
      "language": "php",
      "name": "Select"
    },
    {
      "code": "<?php\ntry {\n  // Connecting to Ignite using pre-configured DSN.\n  $dbh = new PDO('odbc:Apache Ignite DSN');\n\n  // Performing query.\n  $dbh->query('DELETE FROM Person WHERE firstName = \\'John\\' and lastName = \\'Smith\\'');\n\n} catch (PDOException $e) {\n  print \"Error!: \" . $e->getMessage() . \"\\n\";\n  die();\n}\n?>",
      "language": "php",
      "name": "Delete"
    }
  ]
}
[/block]