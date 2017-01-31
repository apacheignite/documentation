* [Overview](#section-overview)
* [Starting Demo Mode](#section-starting-demo-mode)
 * [Configure Clusters and Caches](#section--configure-clusters-and-caches-)
 * [Import Domain Model](#section--import-domain-model-)
 * [Run SQL Queries](#section--run-sql-queries-)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
You can use Ignite Web Console's demo mode to explore and evaluate its various features that help configure and manage an Ignite cluster. In this mode, you can check out predefined clusters, caches, and domain models. An instance of a pre-populated H2 database is started on which you can run various SQL queries and view data charts. You can also monitor various cache and node metrics as well as CPU and Heap usage for the demo cluster.
[block:api-header]
{
  "type": "basic",
  "title": "Starting Demo Mode"
}
[/block]
To enable the demo mode, click on the `Start demo` button located in the top menu of the [Web Console](https://console.gridgain.com/).
[block:callout]
{
  "type": "info",
  "body": "For the simplicity of this guide, an already deployed instance of Ignite Web Console is used. The instance is hosted on GridGain infrastructure and GridGain embeds its logo as a part of the console interface. You may see the logo on the screenshots below. Note that you're free to deploy Ignite Web Console on an [alternative infrastructure](doc:local-deployment) and use other logo.",
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

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/ca768c5-start-ignite-web-agent.png",
        "start-ignite-web-agent.png",
        1356,
        261,
        "#221518"
      ],
      "border": true
    }
  ]
}
[/block]
Once the IgniteWeb Agent is started, you can go back to your web browser where you can:

[block:api-header]
{
  "type": "basic"
}
[/block]
## **Configure Clusters and Caches**
You can click on `Clusters` and `Caches`, on the side bar menu of the web console, to set and view various configurations for Ignite. Click on `Summary` to download these configurations in XML and Java. A ready-to-use Maven based project can also be downloaded from this page.
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
      ],
      "border": true
    }
  ]
}
[/block]
## **Import Domain Model**

In demo mode, an instance of the in-memory H2 database is started on the connected Ignite Web Agent. To evaluate:
  * Go to Ignite Web Console `Domain model` screen.
  * Click `Import from database`. You should see a modal window with the demo description.
  * Click `Next` button to see a list of available schemas.
  * Click `Next` button to see a list of available tables.
  * Click `Next` button to see import options.
  * Select some of them and click `Save`.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/fbf02ed-domain-model.png",
        "domain-model.png",
        1139,
        1063,
        "#a23745"
      ],
      "border": true
    }
  ]
}
[/block]
## **Run SQL Queries**
In this mode, three server and one client nodes will be started. Several caches will be created and populated with data. To evaluate:
 * Click  on `Queries` tab on the Ignite Web Console top menu.
 * `SQL Demo` notebook with preconfigured queries will open.
 * You can execute SQL queries, on the demo database, for tables: `Country, Department, Employee, Parking, Car`.

**Example**
 Enter the following SQL statement:
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
      ],
      "border": true
    }
  ]
}
[/block]