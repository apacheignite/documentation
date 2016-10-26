PHP provides a lightweight, consistent interface for accessing databases named PHP Data Objects   - PDO. This extension goes with several database-specific PDO drivers. One of them is [PDO_ODBC](http://php.net/manual/en/ref.pdo-odbc.php), which allows connecting to any database, which provides ODBC driver.

As Apache Ignite shipped with its own ODBC driver, we have also tested it with ODBC. On this page you can find some hints and tips about Apache Ignite and PDO interoperability.
[block:api-header]
{
  "type": "basic",
  "title": "Getting things working"
}
[/block]
First of all, you need PHP installed on your system with PDO and PDO_ODBC driver. You can find instructions on how you can do it on [PHP website](http://php.net).

The second thing you'll need is an installed Apache Ignite ODBC driver. You may refer to [Getting Started](doc:getting-started) section for details on how to do this.

You also are going to need properly configured DSN for Ignite. In example below we assume your DSN's name is "Apache Ignite DSN".

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