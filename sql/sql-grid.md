In-Memory SQL Grid adds in-memory distributed database capabilities to Apache Ignite. It is horizontally scalable, fault tolerant and SQL ANSI-99 compliant. 

SQL Grid fully supports all SQL and DML commands including SELECT, UPDATE, INSERT, MERGE and DELETE queries. 

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/ee4c650-SQL-Grid-Diagram_v4.png",
        "SQL-Grid-Diagram_v4.png",
        800,
        662,
        "#f86947"
      ]
    }
  ]
}
[/block]
There are several ways to connect to an Ignite cluster and start working with its data. The most generic one is to use common JDBC or ODBC APIs relying on corresponding Ignite's drivers. This approach is truly cross-platform and allows you connecting from analytical tools like Tableau or programming languages like PHP or Ruby. Alternatively, you can work with SQL Grid using Java, .NET and C++ native APIs that were deliberately created for this programming languages.

##Features and Topics
* [Distributed Queries](doc:sql-queries) 
* [Local Queries](doc:local-queries) 
* [Distributed DML](doc:dml) 
* [Indexes](doc:indexes) 
* [Miscellaneous Features](doc:miscellaneous-features) 
* [Configuration Parameters](doc:configuration-parameters) 
* [JDBC Driver](doc:jdbc-driver) 
* [ODBC Driver](doc:odbc-driver) 
* [Geospatial Support](doc:geospatial-queries) 
* [Performance and Debugging](doc:performance-and-debugging)

##SQL Grid Integrations and Tools
* [Appache Zeppelin](https://apacheignite-mix.readme.io/docs/apache-zeppelin)
* [PHP PDO](https://apacheignite-mix.readme.io/docs/php-pdo)