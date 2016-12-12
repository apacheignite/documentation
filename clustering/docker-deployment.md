Docker allows to package Ignite deployment with all the dependencies into a standard container. Docker automates downloading the Ignite release, deploying users' library into Ignite, and configuring nodes. It also automatically starts up a fully configured Ignite node. Such integration allows users to deploy new code by simply restarting the Ignite docker container.
[block:api-header]
{
  "type": "basic",
  "title": "Start Ignite Docker Container"
}
[/block]
For running docker container need to pull and start docker image. By default the latest version is downloaded. Full list of tags here: [https://hub.docker.com](https://hub.docker.com/r/apacheignite/ignite/tags)

To pull the Ignite docker container use the following command:
[block:code]
{
  "codes": [
    {
      "code": "# Pull latest version.\nsudo docker pull apacheignite/ignite\n\n# Pull ignite version {ignite-version}\nsudo docker pull apacheignite/ignite:{ignite-version}",
      "language": "shell"
    }
  ]
}
[/block]
To run Ignite docker container using `docker run`:
[block:code]
{
  "codes": [
    {
      "code": "# Run latest version.\nsudo docker run -it --net=host \n-e \"CONFIG_URI=$CONFIG_URI\" \n[-e \"OPTION_LIBS=$OPTION_LIBS\"]\n[-e \"JVM_OPTS=$JVM_OPTS\"]\n...\napacheignite/ignite \n\n# Run ignite version {ignite-version}\nsudo docker run -it --net=host \n-e \"CONFIG_URI=$CONFIG_URI\" \n[-e \"OPTION_LIBS=$OPTION_LIBS\"]\n[-e \"JVM_OPTS=$JVM_OPTS\"]\n...\napacheignite/ignite:{ignite-version}",
      "language": "shell"
    }
  ]
}
[/block]
The configuration parameters are passed through environment variables in docker container. The following configuration parameters are available:
[block:parameters]
{
  "data": {
    "h-0": "Name",
    "h-1": "Description",
    "h-2": "Default",
    "h-3": "Example",
    "0-0": "`CONFIG_URI`",
    "1-0": "`OPTION_LIBS`",
    "2-0": "`JVM_OPTS`",
    "3-0": "`EXTERNAL_LIBS`",
    "0-1": "URL to the Ignite configuration file (can also be relative to the  META-INF folder on the class path). The downloaded config file will be saved to ./ignite-config.xml",
    "1-1": "Ignite optional libs which will be included in the class path.",
    "2-1": "Environment variables passed to Ignite instance in your docker command.",
    "3-1": "List of URL's to libs.",
    "0-2": "`N/A`",
    "1-2": "`ignite-log4j,\nignite-spring,\nignite-indexing`",
    "2-2": "`N/A`",
    "3-2": "`N/A`",
    "0-3": "`https://raw.githubusercontent.com/apache/ignite/\nmaster/examples/config/example-cache.xml`",
    "1-3": "`ignite-aws,ignite-aop`",
    "2-3": "`-Xms1g -Xmx1g -server -XX:+AggressiveOpts -XX:MaxPermSize=256m`",
    "3-3": "`http://central.maven.org/maven2/io/undertow/undertow-servlet/1.3.10.Final/undertow-servlet-1.3.10.Final.jar,http://central.maven.org/maven2/io/undertow/undertow-build-config/1.0.0.Beta24/undertow-build-config-1.0.0.Beta24.jar`"
  },
  "cols": 4,
  "rows": 4
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
To run Ignite docker container, use the following command:
[block:code]
{
  "codes": [
    {
      "code": "sudo docker run -it --net=host -e \"IGNITE_CONFIG=https://raw.githubusercontent.com/apache/ignite/master/examples/config/example-cache.xml\" apacheignite/ignite",
      "language": "shell"
    }
  ]
}
[/block]
Should see the following in logs:
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/ryYtMcSCuGiyVcXN1GCw_dock_git_repo.png",
        "dock_git_repo.png",
        "979",
        "643",
        "#6098ae",
        ""
      ]
    }
  ]
}
[/block]