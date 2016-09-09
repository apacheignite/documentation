* [Overview](#overview)
* [Demo Mode](#demo-mode)
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Ignite Web Console is an interactive configuration wizard and management tool that allows you to:
* Create and download various configurations for Apache Ignite cluster.
* Automatically load SQL metadata from any RDBMS
* Provides management capabilities which allow you to run SQL queries
 on your in-memory cache as well as view execution plans, in-memory schema, and streaming charts.
[block:api-header]
{
  "type": "basic",
  "title": "Demo Mode"
}
[/block]
In order to simplify evaluation of Ignite Web Console demo mode was implemented.
To start demo, you need to click button "Start demo". New tab will be open with prepared demo data on each screen.
## Demo for import domain model from database
In this mode an in-memory H2 database will be started on connected Ignite Web Agent.
How to evaluate:
  * Go to Ignite Web Console "Domain model" screen.
  * Click "Import from database". You should see modal with demo description.
  * Click "Next" button. You should see list of available schemas.
  * Click "Next" button. You should see list of available tables.
  * Click "Next" button. You should see import options.
  * Select some of them and click "Save".