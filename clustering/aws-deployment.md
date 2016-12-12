The Apache Ignite AMI (Amazon Machine Image) allows you to set up a simple Apache Ignite cluster using the Amazon Web Services EC2 Management Console. Installing via the AMI allows you to quickly deploy a Apache Ignite cluster.
[block:api-header]
{
  "type": "basic",
  "title": "Amazon EC2 Deployment"
}
[/block]
1. Choose the required region and click on link in table below.
[block:parameters]
{
  "data": {
    "0-0": "`US-WEST`",
    "1-0": "`US-EAST`",
    "2-0": "`EU-CENTRAL`",
    "0-1": "[ami-9cdbb3fc](https://console.aws.amazon.com/ec2/home?region=us-west-1#launchAmi=ami-9cdbb3fc)",
    "1-1": "[ami-ce82caa4](https://console.aws.amazon.com/ec2/home?region=us-east-1#launchAmi=ami-ce82caa4)",
    "2-1": "[ami-191b0775](https://console.aws.amazon.com/ec2/home?region=eu-central-1#launchAmi=ami-191b0775)",
    "h-0": "Region",
    "h-1": "Image"
  },
  "cols": 2,
  "rows": 3
}
[/block]
or search image in `Community AMIs` by `Apache Ignite` keywords:
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/faf5b14-search.png",
        "search.png",
        816,
        443,
        "#eaeae7"
      ]
    }
  ]
}
[/block]
2. Choose an `Instance Type`.
3. Go to `Configure Instance` and expand `Advanced Details` section.
4. Add any of the following configuration parameters:
[block:parameters]
{
  "data": {
    "0-0": "`CONFIG_URI`",
    "1-0": "`OPTION_LIBS`",
    "2-0": "`JVM_OPTS`",
    "3-0": "`EXTERNAL_LIBS`",
    "h-0": "Name",
    "h-1": "Description",
    "h-2": "Default",
    "h-3": "Example",
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
    "3-3": "`http://central.maven.org/maven2/io/undertow/undertow-servlet/1.3.10.Final/undertow-servlet-1.3.10.Final.jar,http://central.maven.org/maven2/io/undertow/undertow-build-config/1.0.0.Beta24/undertow-build-config-1.0.0.Beta24.jar`",
    "4-0": "`IGNITE_VERSION`",
    "4-1": "Version of Apache Ignite",
    "4-2": "`latest'",
    "4-3": "1.7.0"
  },
  "cols": 4,
  "rows": 5
}
[/block]

[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/0c08e64-advance_details.png",
        "advance_details.png",
        882,
        502,
        "#dadbd3"
      ]
    }
  ]
}
[/block]
5. On the Tag Instance, set the value for `Name` tag. For example `ignite-node`
6. Review and run instances.
7. Connect to the instances [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html)
8. To access the execution progress, you need to know the `container id`. Use the following command:
[block:code]
{
  "codes": [
    {
      "code": "sudo docker ps",
      "language": "shell"
    }
  ]
}
[/block]
9. Show logs:
[block:code]
{
  "codes": [
    {
      "code": "sudo docker logs -f CONTAINER_ID",
      "language": "shell"
    }
  ]
}
[/block]
10. Enter the docker container:
[block:code]
{
  "codes": [
    {
      "code": "sudo docker exec -it container_id /bin/bash",
      "language": "shell"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": ""
}
[/block]