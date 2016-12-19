* [Prerequisites](#prerequisites)
* [Building Ignite Web Agent](#building-ignite-web-agent)
* [Run Ignite Web Console Frontend](#run-ignite-web-console-in-development-mode)
[block:api-header]
{
  "type": "basic",
  "title": "Prerequisites"
}
[/block]
In order to deploy Ignite Web Console locally, you should install:

* MongoDB (version >=3.2.x) follow instructions from site [http://docs.mongodb.org/manual/installation](http://docs.mongodb.org/manual/installation)
* NodeJS (version >=6.5.x) using installer from site [https://nodejs.org/en/download/current](https://nodejs.org/en/download/current) for your OS.
*Change directory '$IGNITE_HOME/modules/web-console/src/main/js'.
*Run "npm install --no-optional" in terminal for download dependencies.
[block:api-header]
{
  "type": "basic",
  "title": "Building Ignite Web Agent"
}
[/block]
To build Ignite Web Agent from sources, run the following command from `$IGNITE_HOME` folder:
`mvn clean package -pl :ignite-web-agent -am -P web-console -DskipTests=true`

Once the build process is over, you can find `ignite-web-agent-x.x.x.zip` in: 
`$IGNITE_HOME/modules/web-agent/target`
[block:api-header]
{
  "type": "basic",
  "title": "Run Ignite Web Console In Development Mode"
}
[/block]
To run Ignite Web Console in development mode, you need to:
* Configure MongoDB to run as service or in terminal start MongoDB by executing `mongod` command
* Copy `ignite-web-agent-x.x.x.zip` to `$IGNITE_HOME/modules/web-console/src/main/js/serve/agent_dists` folder
* In new terminal change directory to `$IGNITE_HOME/modules/web-console/src/main/js`.
If needed run `npm install --no-optional` (if dependencies changed) and run `node serve` to start backend
* In new terminal change directory to '$IGNITE_HOME/modules/web-console/src/main/js'.
If needed run `npm install --no-optional` (if dependencies changed) and start webpack in development mode `npm run dev`
* In browser open: http://localhost:9000