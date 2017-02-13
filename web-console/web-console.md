[Ignite Web Console](https://ignite.apache.org/addons.html#web-console) is an interactive configuration wizard, management and monitoring tool that allows you to:
* Create and download various configurations to use for your Apache Ignite cluster.

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/f5eeab2-cluster.png",
        "cluster.png",
        1155,
        1220,
        "#efeeec"
      ],
      "border": true,
      "sizing": "smart"
    }
  ]
}
[/block]
* Automatically construct Apache Ignite's SQL metadata from any RDBMS schemas.

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/16d8b74-model.png",
        "model.png",
        1159,
        1073,
        "#f0f0ef"
      ],
      "border": true,
      "sizing": "smart"
    }
  ]
}
[/block]
* Execute SQL queries over your in-memory caches and view the execution plans.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/0383009-Screen_Shot_2017-02-02_at_12.47.09_PM.png",
        "Screen Shot 2017-02-02 at 12.47.09 PM.png",
        1714,
        1406,
        "#5d9cc3"
      ],
      "border": true,
      "sizing": "smart"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "For the simplicity of this guide, an [already deployed instance](https://console.gridgain.com) of Ignite Web Console is used. The instance is hosted on GridGain infrastructure and GridGain embeds its logo as a part of the console interface. You may see the logo on the screenshots throughout this guide. Note that you're free to deploy Ignite Web Console on an [alternative infrastructure](doc:local-deployment) and use another logo.",
  "title": "Web Console Hosting and Logo"
}
[/block]
Ignite ships with **Ignite Web console** - a web application that can be deployed on your system environment. It allows configuring all the cluster properties and import schema from a database for integrating with persistence stores. It can connect to the specified database and generate all the required OR-mapping configuration (XML and pure Java) and Java domain model POJOs. The web console also features cluster monitoring functionality (available separately as GridGain plugin) that shows various cache and node metrics as well as CPU and heap usage. 

Refer to the following pages to learn more:

* [Ignite Web Agent](doc:web-agent) 
* [Local Deployment](doc:local-deployment)
* [Hosted Deployment](doc:hosted-deployment) 
* [Cluster Configuration](doc:webconsole-cluster-configuration) 
* [Queries Execution](doc:webconsole-queries-execution) 
* [Demo Mode](doc:demo-mode) 
* [RDBMS Integration](https://apacheignite-mix.readme.io/docs/web-console)