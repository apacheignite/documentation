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
* [Enable](http://php.net/manual/en/ref.pdo-odbc.php) PDO_ODBC driv
er.
* If necessary, [configure](php.net/manual/en/ref.pdo-odbc.php) PDO_ODBC driver for your system.
[block:api-header]
{
  "type": "basic",
  "title": "Starting Ignite Cluster"
}
[/block]
After PHP PDO is installed and ready to be used let's start an Ignite cluster with an exemplary configuration and connect to the cluster from a PHP application updating and querying cluster's data.

TBD: Let's stick to an example where we will start a single node cluster with one cache pre-configured. The cache configuration must contain QueryEntities with key and value fields that will be used in SQL queries (INSERT, UPDATE, DELETE, SELECT). 

[block:code]
{
  "codes": [
    {
      "code": "CONFIGURATION THAT IS USED in the example",
      "language": "xml"
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