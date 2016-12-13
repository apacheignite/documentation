The Apache Ignite image allows you to set up a simple Apache Ignite cluster using the Google Compute Console. Installing via the image allows you to quickly deploy a Apache Ignite cluster.
[block:api-header]
{
  "type": "basic",
  "title": "Google Compute Deployment"
}
[/block]
1. To import the [Ignite Image](https://storage.googleapis.com/ignite-media/ignite-google-image.tar.gz), execute the following command:
[block:code]
{
  "codes": [
    {
      "code": "gcloud compute images create ignite-image \\\n   --source-uri gs://ignite-media/ignite-google-image.tar.gz\n",
      "language": "shell"
    }
  ]
}
[/block]
For information please refer to [cloud.google.com](https://cloud.google.com/compute/docs/images#import_an_image)

2. Go to `Google Compute Console`.
3. Go to `Compute->Compute Engine->VM` instances and click on `New instance`.
4. Click on `Change` button in section `Boot disk`.
5. Go to `Custom images` and choose imported image. On the screenshot the image is with `ignite-name` name.
[block:image]
{
  "images": [
    {
      "image": [
        "https://files.readme.io/e02e235-choose_image.png",
        "choose_image.png",
        684,
        478,
        "#e4e7ea"
      ]
    }
  ]
}
[/block]
6. Click on `Management, disk, networking, access & security options` and add any of the following configuration parameters.
[block:parameters]
{
  "data": {
    "h-0": "Name",
    "h-1": "Description",
    "h-2": "Default",
    "h-3": "Example",
    "0-0": "`CONFIG_URI`",
    "0-1": "URL to the Ignite configuration file (can also be relative to the  META-INF folder on the class path). The downloaded config file will be saved to ./ignite-config.xml",
    "0-2": "`N/A`",
    "1-0": "`OPTION_LIBS`",
    "1-1": "Ignite optional libs which will be included in the class path.",
    "1-2": "`ignite-log4j,\nignite-spring,\nignite-indexing`",
    "2-0": "`JVM_OPTS`",
    "2-1": "Environment variables passed to Ignite instance in your docker command.",
    "2-2": "`N/A`",
    "3-0": "`EXTERNAL_LIBS`",
    "3-1": "List of URL's to libs.",
    "3-2": "`N/A`",
    "4-0": "`IGNITE_VERSION`",
    "4-1": "Version of Apache Ignite",
    "4-2": "`latest'",
    "4-3": "1.7.0",
    "3-3": "`http://central.maven.org/maven2/io/undertow/undertow-servlet/1.3.10.Final/undertow-servlet-1.3.10.Final.jar,http://central.maven.org/maven2/io/undertow/undertow-build-config/1.0.0.Beta24/undertow-build-config-1.0.0.Beta24.jar`",
    "2-3": "`-Xms1g -Xmx1g -server -XX:+AggressiveOpts -XX:MaxPermSize=256m`",
    "1-3": "`ignite-aws,ignite-aop`",
    "0-3": "`https://raw.githubusercontent.com/apache/ignite/\nmaster/examples/config/example-cache.xml`"
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
        "https://files.readme.io/1945104-metadata.png",
        "metadata.png",
        734,
        482,
        "#487bd4"
      ]
    }
  ]
}
[/block]

[block:callout]
{
  "type": "success",
  "title": "Preferred Ignite Version",
  "body": "IGNITE_VERSION attribute can vary depending on an Apache Ignite version of interest."
}
[/block]
7. Fill the required fields and run instances.
8. Connect to the instances.
9. To access the execution progress you need to know a `container id`. The following command will show the `container id`:
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
10. Show logs:
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
 11. Enter the docker container:
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