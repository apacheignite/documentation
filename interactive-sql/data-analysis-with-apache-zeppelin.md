* [Overview](#overview)
* [Zeppelin Installation and Configuration](#zeppelin-installation-and-configuration)
* [Configuring Ignite Interpreters](#configuring-ignite-interpreters)
* [Using Ignite Interpreters](##using-ignite-interpreters)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
[Apache Zeppelin](http://zeppelin.incubator.apache.org), a web-based notebook that enables interactive data analytics. You can make beautiful data-driven, interactive and collaborative documents with SQL, Scala and more. 

You can use Zeppelin to retrieve distributed data from cache using Ignite SQL interpreter. Moreover, Ignite interpreter allows you to execute any Scala code in cases when SQL doesn't fit to your requirements. For example you can populate data into your caches or execute distributed computations.
[block:api-header]
{
  "type": "basic",
  "title": "Zeppelin Installation and Configuration"
}
[/block]
In order to start using Ignite interpreters you should install Zeppelin in two simple steps: 
  1. Clone Zeppelin Git repository
  `git clone https://github.com/apache/incubator-zeppelin.git`
  2. Build Zeppelin from sources
  `cd incubator-zeppelin`
  `mvn clean install -Dignite-version=1.2.0-incubating -DskipTests`
[block:callout]
{
  "type": "success",
  "title": "Building Zeppelin with specific Ignite version",
  "body": "You can use `ignite-version` property for build Zeppelin with specific Ignite version. Use version `1.1.0-incubating` or later."
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "By default Ignite and Ignite SQL interpreters are already configured in Zeppelin. Otherwise you should add the following interpreters class names to the corresponding configuration file or environment variable (see \"Configure\" section of [Zeppelin installation guide](https://zeppelin.incubator.apache.org/docs/0.5.5-incubating/install/install.html)): \n  * `org.apache.zeppelin.ignite.IgniteInterpreter`    \n  * `org.apache.zeppelin.ignite.IgniteSqlInterpreter`\n\n**Note:** First interpreter become a default.",
  "title": "Adding Ignite Interpreters"
}
[/block]
Once Zeppelin are installed and configured you can start it using command 
`./bin/zeppelin-daemon.sh start`  
and open start page in your browser (default start page URL is [http://localhost:8080](http://localhost:8080)).
[block:image]
{
  "images": [
    {
      "caption": "Apache Zeppelin start page",
      "image": [
        "https://www.filepicker.io/api/file/3bHTJnpSvGiI2KEUIM6L",
        "Selection_001.png",
        "1142",
        "618",
        "#346d9f",
        ""
      ]
    }
  ]
}
[/block]
See also [Zeppelin installation documentation](https://zeppelin.incubator.apache.org/docs/0.5.5-incubating/install/install.html).
[block:api-header]
{
  "type": "basic",
  "title": "Configuring Ignite Interpreters"
}
[/block]
Click on "Interpreter" menu item. This page contains settings for all configured interpreter groups. Scroll down to the "Ignite" section and modify properties as you need using "Edit" button. Click "Save" button to save changes in configuration. Don't forget restart interpreter after changes in configuration.
[block:image]
{
  "images": [
    {
      "caption": "Apache Ignite interpreters settings",
      "image": [
        "https://www.filepicker.io/api/file/vOhUlN7XRXiE0Gf5bPgC",
        "Selection_002.png",
        "1142",
        "671",
        "#3571a4",
        ""
      ]
    }
  ]
}
[/block]
## Configuring Ignite SQL Interpreter
Ignite SQL interpreter requires only `ignite.jdbc.url` property that contains JDBC connection URL. In our example we will use `words` cache. Edit `ignite.jdbc.url` property setting the following value: `jdbc:ignite://localhost:11211/words`.

See also [JDBC Driver](http://apacheignite.readme.io/v1.2/docs/jdbc-driver)  section for details.
 
## Configuring Ignite Interpreter
For most simple cases Ignite interpreter requires the following properties:

  * `ignite.addresses` - Coma separated list of Ignite cluster hosts. See [Cluster Configuration](http://apacheignite.readme.io/v1.2/docs/cluster-config) section for details.
  * `ignite.clientMode` - You can connect to the Ignite cluster as client or server node. See [Clients vs. Servers](http://apacheignite.readme.io/v1.2/docs/clients-vs-servers) section for details. Use `true` or `false` values in order to connect in client or server mode respectively.
  * `ignite.peerClassLoadingEnabled` - Enables peer-class-loading. See [Zero Deployment](http://apacheignite.readme.io/v1.2/docs/zero-deployment) section for details. Use `true` or `false` values in order to enable or disable P2P class loading respectively.

For more complicated cases you can define own configuration of Ignite using `ignite.config.url` property that contains URL to Ignite configuration file. Note that if `ignite.config.url` property is defined then all aforementioned properties will be ignored.
[block:api-header]
{
  "type": "basic",
  "title": "Using Ignite Interpreters"
}
[/block]
## Starting Ignite cluster
Before using Zeppelin we need start Ignite cluster. Download [Apache Ignite In-Memory Data Fabric binary release](https://ignite.apache.org/download.cgi#binaries) and unpack the downloaded archive:
`unzip apache-ignite-fabric-1.6.0-bin.zip -d <dest_dir>`

Examples are shipped as a separate Maven project, so to start running you simply need
to import provided `<dest_dir>/apache-ignite-fabric-1.6.0-bin/pom.xml` file into your favourite IDE.

Start the following examples:
  * `org.apache.ignite.examples.ExampleNodeStartup` - starts one Ignite node. You can start one or more nodes.
  * `org.apache.ignite.examples.streaming.wordcount.StreamWords` - starts client node that continuously streams words into `words` cache.

Now you are ready for using Zeppelin for accesing to our Ignite cluster.

## Creating new note in Zeppelin
Create new (or open existing) note using "Notebook" menu item. 
[block:image]
{
  "images": [
    {
      "caption": "Creating new note",
      "image": [
        "https://www.filepicker.io/api/file/pF9Q8948SeOG7GPtyH0v",
        "Selection_003.png",
        "1137",
        "662",
        "#366fa2",
        ""
      ]
    }
  ]
}
[/block]
After creating new note you should click "Notebook" menu item again and open created note. Click to note's name in order to rename it. Enter new title and press "Enter" key.
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/Maj9hwbjSpi0GrTDqfeE",
        "Selection_004.png",
        "1157",
        "232",
        "#3474ac",
        ""
      ],
      "caption": "New note"
    }
  ]
}
[/block]
Since note is created you can input SQL query or Scala code and execute it clicking to "Execute" button (blue triangle icon).
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/oOQjImXzTlGlImyEwbxx",
        "Selection_005.png",
        "1158",
        "234",
        "#3474ac",
        ""
      ],
      "caption": "New note with user defined name"
    }
  ]
}
[/block]
## Using Ignite SQL interpreter
For execute SQL query use `%ignite.ignitesql` prefix and your SQL query. For example we can select top 10 words in our `words` cache using the following query:
[block:code]
{
  "codes": [
    {
      "code": "%ignite.ignitesql select _val, count(_val) as cnt from String group by _val order by cnt desc limit 10",
      "language": "sql"
    }
  ]
}
[/block]

[block:image]
{
  "images": [
    {
      "caption": "Using Ignite SQL interpreter",
      "image": [
        "https://www.filepicker.io/api/file/NRcgOAoyT0OXinq2qpeb",
        "Selection_006.png",
        "1156",
        "257",
        "#3474ac",
        ""
      ]
    }
  ]
}
[/block]
After executing this example you can see result as table or graph. Use corresponding icons to change view.
[block:image]
{
  "images": [
    {
      "caption": "SQL query result as table",
      "image": [
        "https://www.filepicker.io/api/file/0xedliKRxinxivxgHwcC",
        "Selection_007.png",
        "1143",
        "640",
        "#336da1",
        ""
      ]
    }
  ]
}
[/block]

[block:image]
{
  "images": [
    {
      "caption": "SQL query result as graph",
      "image": [
        "https://www.filepicker.io/api/file/7ZwjGd0ZTUm02yY3Mmig",
        "Selection_008.png",
        "1157",
        "551",
        "#549bc4",
        ""
      ]
    }
  ]
}
[/block]

[block:image]
{
  "images": [
    {
      "caption": "SQL query result as pie chart",
      "image": [
        "https://www.filepicker.io/api/file/vGib202NTiaQkpxyvH5q",
        "Selection_009.png",
        "1141",
        "650",
        "#3074ad",
        ""
      ]
    }
  ]
}
[/block]
## Using Ignite interpreter
For execute Scala code snippet use `%ignite` prefix and your code snippet. For example we can select average, min and max counts among all the words:
[block:code]
{
  "codes": [
    {
      "code": "%ignite\nimport org.apache.ignite._\nimport org.apache.ignite.cache.affinity._\nimport org.apache.ignite.cache.query._\nimport org.apache.ignite.configuration._\n\nimport scala.collection.JavaConversions._\n\nval cache: IgniteCache[AffinityUuid, String] = ignite.cache(\"words\")\n\nval qry = new SqlFieldsQuery(\"select avg(cnt), min(cnt), max(cnt) from (select count(_val) as cnt from String group by _val)\", true)\n\nval res = cache.query(qry).getAll()\n\ncollectionAsScalaIterable(res).foreach(println _)",
      "language": "scala"
    }
  ]
}
[/block]

[block:image]
{
  "images": [
    {
      "caption": "Using Ignite interpreter",
      "image": [
        "https://www.filepicker.io/api/file/NdEH40jSLGaEML8LD5lw",
        "Selection_010.png",
        "1156",
        "473",
        "#3474ac",
        ""
      ]
    }
  ]
}
[/block]
After executing this example you will see output of Scala REPL.
[block:image]
{
  "images": [
    {
      "caption": "Scala REPL output",
      "image": [
        "https://www.filepicker.io/api/file/DsaHrKE3QKCPHNSVSmLO",
        "Selection_011.png",
        "1143",
        "649",
        "#3474ac",
        ""
      ]
    }
  ]
}
[/block]

[block:callout]
{
  "type": "warning",
  "body": "Note that Ignite version of your Ignite cluster and Zeppelin installation must be equal."
}
[/block]