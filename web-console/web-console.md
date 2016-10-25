* [Overview](#overview)
* [Demo Mode](#demo-mode)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite Web Console is an interactive configuration wizard, management and monitoring tool that allows you to:
* Create and download various configurations to use for your Apache Ignite cluster.
* Automatically construct Apache Ignite's SQL metadata from any RDBMS schemas.
* Execute SQL queries over your in-memory caches as well as view their execution plans, in-memory schemas and streaming charts.
[block:api-header]
{
  "type": "basic",
  "title": "Demo Mode"
}
[/block]
You can use Web Console's demo mode for evaluation purposes. To enable this mode, you need to click on the `Start demo` button located in the top menu, and wait for a popup screen to appear that provides additional steps.

## Importing a domain model from a database
In this mode, an instance of in-memory H2 database will be started on the connected Ignite Web Agent.
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