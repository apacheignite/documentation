[Ignite Web Console](https://console.gridgain.com/) is an interactive configuration wizard, management and monitoring tool that allows you to:
* Create and download various configurations to use for your Apache Ignite cluster.
* Automatically construct Apache Ignite's SQL metadata from any RDBMS schemas.
* Execute SQL queries over your in-memory caches.
*  View query execution plans, in-memory schemas and streaming charts.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/d387efa-monitoring.png",
        "monitoring.png",
        1145,
        1201,
        "#f1e9e3"
      ],
      "border": true
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "For the simplicity of this guide, an already deployed instance of Ignite Web Console is used. The instance is hosted on GridGain infrastructure and GridGain embeds its logo as a part of the console interface. You may see the logo on the screenshots through this guide. Note that you're free to deploy Ignite Web Console on an [alternative infrastructure](doc:local-deployment) and use other logo.",
  "title": "Web Console Hosting and Logo"
}
[/block]
Ignite ships with **Ignite Web console**, a web application, that can be deployed on your system environment. Ignite Web Console that allows configuring all the cluster properties and import schema from a database for integrating with persistence stores. It can connect to the specified database and generate all the required OR-mapping configuration (XML and pure Java) and Java domain model POJOs.

* [Demo Mode](doc:demo-mode) 
* [Local Deployment](doc:local-deployment)
* [Web Agent](doc:web-agent) 
* [RDBMS Integration](https://apacheignite-mix.readme.io/docs/web-console)