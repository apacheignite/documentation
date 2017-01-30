* [Starting Demo Mode](#section-starting-demo-mode)
* [Importing a domain model from a database](#section-importing-a-domain-model-from-a-database)
* [SQL Queries](#section-sql-queries)

[block:api-header]
{
  "type": "basic",
  "title": "Starting Demo Mode"
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
2. Open the command shell and, assuming you are in `ignite-web-agent` directory, run the following command: 
[block:code]
{
  "codes": [
    {
      "code": "./ignite-web-agent.sh",
      "language": "shell"
    }
  ]
}
[/block]
3. Once the IgniteWeb Agent is started, you can go back to your web browser where you can:
## Checkout predefined cluster and caches
You can click on **Clusters** and **Caches**, on the side bar menu of the web console, to set and view various configurations for Ignite. Click on **Summary** to download these configurations in XML, and Java. A ready-to-use Maven based project can also be downloaded from this page.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/857edc8-summary.png",
        "summary.png",
        1141,
        1168,
        "#eeebe9"
      ]
    }
  ]
}
[/block]
## Import Domain Model From Database

In this mode, an instance of the in-memory H2 database will be started on the connected Ignite Web Agent. To evaluate:
  * Go to Ignite Web Console `Domain model` screen.
  * Click `Import from database`. You should see a modal window with the demo description.
  * Click `Next` button. You should see list of available schemas.
  * Click `Next` button. You should see list of available tables.
  * Click `Next` button. You should see import options.
  * Select some of them and click `Save`.
[block:image]
{
  "images": [
    {
      "image": []
    }
  ]
}
[/block]
## Run SQL queries on the demo database 
In this mode, three server and one client nodes will be started. Several caches will be created and populated with data. To evaluate:
 * Click `Queries` in Ignite Web Console top menu.
 * `SQL Demo` notebook with preconfigured queries will open.
 * You can execute any SQL queries for tables: `Country, Department, Employee, Parking, Car`.

**Example**
 * Enter the following SQL statement:
[block:code]
{
  "codes": [
    {
      "code": "SELECT p.name, count(*) AS cnt FROM \"ParkingCache\".Parking p`\n`INNER JOIN \"CarCache\".Car c ON (p.id) = (c.parkingId)`\n`GROUP BY P.NAME",
      "language": "text"
    }
  ]
}
[/block]
* Click the `Execute` button. You should get some data in the table.
* Click `charts` buttons to see auto generated charts.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/5b5cdc8-sql-queries.png",
        "sql-queries.png",
        1147,
        1195,
        "#539ac3"
      ]
    }
  ]
}
[/block]
## Monitor the cluster and view various heap, CPU, and other useful node and cache metrics.
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
      ],
      "border": true
    }
  ]
}
[/block]




[block:code]
{
  "codes": [
    {
      "code": "",
      "language": "text"
    }
  ]
}
[/block]