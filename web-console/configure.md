* [Clusters](#section-clusters)
* [Model](#section-model)
* [Caches](#section-caches)
* [IGFS](#section-igfs)
* [Summary](section-summary) 
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Under the **Configure** tab of the Web Console, you can efficiently create configuration files and code snippets for your Apache Ignite projects. You can configure Ignite clusters, caches, import domain model from any RDBMS, that supports JDBC driver, and generate OR-mapping configuration and POJO's. 
[block:api-header]
{
  "type": "basic",
  "title": "Clusters"
}
[/block]
Using the Ignite Web Console, you can set various general and advanced level configurations for your Ignite cluster. The Web Console creates these configurations in Spring XML and Java, which can be downloaded and copied into your project for your convenience.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/b6eb965-cluster.png",
        "cluster.png",
        1155,
        1220,
        "#efeeec"
      ],
      "border": true
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Model"
}
[/block]
To speed the creation of your configuration files, the Ignite Web Console allows you to connect to the database and import schemas, configure indexed types, and automatically generate all the required XML OR-mapping configuration and Java domain model POJOs. Ignite can integrate with any RDBMS that supports JDBC driver - Oracle, PostgreSQL, Microsoft SQL Server, and MySQL.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/29e26b5-model.png",
        "model.png",
        1159,
        1073,
        "#f0f0ef"
      ],
      "border": true
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Caches"
}
[/block]
The Ignite Web Console facilitates creating and configuring Ignite caches. You can configure memory settings, persistence, as well as various advanced level settings for multiple caches associated with your Ignite cluster.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/d8d186d-caches.png",
        "caches.png",
        1151,
        1037,
        "#f0f0ef"
      ],
      "border": true
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "IGFS"
}
[/block]
You can you configure Ignite In-Memory File System that allows you to work with files and directories over existing cache infrastructure. IGFS can either work as purely in-memory file system, or delegate to another file system (e.g. various Hadoop file system implementations) acting as a caching layer. Additionally, IGFS provides API to execute map-reduce tasks over file system data.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/3eab31c-igfs.png",
        "igfs.png",
        1149,
        1051,
        "#f0f0f0"
      ],
      "border": true
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Summary"
}
[/block]
Using the Web Console Summary feature, you can download a ready-to-use Maven based project that contains Ignite configurations in XML and Java, as well as JAVA domain model POJO's. You can also copy these configurations and POJO's into your existing project. The Web Console also generates a Docker file with instructions to create an Apache Ignite Docker image.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/f2db4d6-summary.png",
        "summary.png",
        1141,
        1168,
        "#eeebe9"
      ],
      "border": true
    }
  ]
}
[/block]