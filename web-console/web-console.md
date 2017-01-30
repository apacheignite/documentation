* [Overview](#overview)
* [Demo Mode](#demo-mode)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
[Ignite Web Console](https://console.gridgain.com/) is an interactive configuration wizard, management and monitoring tool that allows you to:
* Create and download various configurations to use for your Apache Ignite cluster.
* Automatically construct Apache Ignite's SQL metadata from any RDBMS schemas.
* Execute SQL queries over your in-memory caches.
*  View query execution plans, in-memory schemas and streaming charts.
[block:api-header]
{
  "type": "basic",
  "title": "Demo Mode"
}
[/block]
You can use the Web Console's demo mode for evaluation purposes. To enable this mode, you need to click on the `Start demo` button located in the top menu, and wait for a popup screen to appear that provides additional steps.
[block:callout]
{
  "type": "info",
  "body": "For the simplicity of this guide, an already deployed instance of Ignite Web Console is used. The instance is hosted on GridGain infrastructure and GridGain embeds its logo as a part of the console interface. You'll see the logo on the screenshots below. Note that you're free to deploy Ignite Web Console on an [alternative infrastructure](doc:local-deployment) and use other logo.",
  "title": "Web Console Hosting and Logo"
}
[/block]
1. Download and unzip Ignite Web Agent. 
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/037ccba-download-web-agent.png",
        "download-web-agent.png",
        691,
        399,
        "#ebebeb"
      ]
    }
  ]
}
[/block]
2. Open the command shell and, assuming you are in Ignite Web Agent installation folder, run the following command: 
[block:code]
{
  "codes": [
    {
      "code": "./ignite-web-agent.sh",
      "language": "text"
    }
  ]
}
[/block]
Once the IgniteWeb Agent is started, you can monitor the cluster and view various heap, CPU, and other useful node and cache metrics.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/79a6424-monitoring.png",
        "monitoring.png",
        1145,
        1201,
        "#f1e9e3"
      ]
    }
  ]
}
[/block]
## Importing a domain model from a database
In this mode, an instance of the in-memory H2 database will be started on the connected Ignite Web Agent.
How to evaluate:
  * Go to Ignite Web Console `Domain model` screen.
  * Click `Import from database`. You should see a modal window with the demo description.
  * Click `Next` button. You should see list of available schemas.
  * Click `Next` button. You should see list of available tables.
  * Click `Next` button. You should see import options.
  * Select some of them and click `Save`.

## SQL Demonstration
How to evaluate:
In this mode, three server and one client nodes will be started. Several caches will be created and populated with data.
 * Click `SQL` in Ignite Web Console top menu.
 * `Demo` notebook with preconfigured queries will be opened.
 * You can execute any SQL queries for tables: `Country, Department, Employee, Parking, Car`.

For example:
 * Enter SQL statement:
`SELECT p.name, count(*) AS cnt FROM "ParkingCache".Parking p`
`INNER JOIN "CarCache".Car c ON (p.id) = (c.parkingId)`
`GROUP BY P.NAME`
 * Click `Execute` button. You should get some data in table.
 * Click `charts` buttons to see auto generated charts.