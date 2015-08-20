Docker allows to package Ignite deployment with all the dependencies into a standard container. Docker automates downloading the Ignite release, deploying users' code into Ignite, and configuring nodes. It also automatically starts up a fully configured Ignite node.

Ignite docker container can run it two modes:  
[block:api-header]
{
  "type": "basic",
  "title": "Start from User Git Repository"
}
[/block]
Ignite docker container will start in this mode if  the `GIT_REPO` parameter is configured. In this case, the container will build user's project specified by the GIT repository and will start Ignite with user code deployed in it. Such integration allows users to deploy new code by simply restarting the Ignite docker container.

To pull the Ignite docker container use the following command:
[block:code]
{
  "codes": [
    {
      "code": " sudo docker pull apacheignite/ignite-docker",
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
    "h-2": "Optional",
    "h-3": "Default",
    "h-4": "Example",
    "0-0": "`GIT_REPO`",
    "1-0": "`GIT_BRANCH`",
    "2-0": "`BUILD_CMD`",
    "0-1": "URL to the GIT repository.",
    "1-1": "GIT branch to build.",
    "2-1": "Command which will be used to build the GIT project.",
    "0-2": "`false`",
    "1-2": "`true`",
    "2-2": "`true`",
    "0-3": "`N/A`",
    "1-3": "`master`",
    "2-3": "`mvn clean package`",
    "0-4": "`https://github.com/bob/ignite-pojo`",
    "1-4": "`sprint-1`",
    "2-4": "`mvn clean package` \\\n`-DskipTests=true`",
    "3-2": "`true`",
    "3-0": "`IGNITE_CONFIG`",
    "3-1": "URL to the Ignite configuration file (can also be relative to the  META-INF folder on the class path). The downloaded config file will be saved to ./ignite-config.xml",
    "3-3": "`N/A`",
    "3-4": "`https://raw.githubusercontent.com/`\n`bob/master/ignite-cfg.xml`",
    "4-0": "`LIB_PATTERN`",
    "4-1": "If set then Ignite docker container will only copy the files which match this regex pattern.",
    "4-2": "`true`",
    "4-3": "`copy all jar files`\n`from target folder`",
    "5-0": "`OPTION_LIBS`",
    "5-1": "Ignite optional libs which will be included in the class path.",
    "5-2": "`true`",
    "5-3": "`ignite-log4j,` \\\n`ignite-spring,` \\\n`ignite-indexing`",
    "5-4": "`ignite-aws,ignite-aop`",
    "4-4": "`libs/.*`"
  },
  "cols": 5,
  "rows": 6
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
To run Ignite docker container from GIT repository, execute the following command:
[block:code]
{
  "codes": [
    {
      "code": "sudo docker run -it -e --net=host \\\n\"GIT_REPO=https://github.com/TikhonovNikolay/docker-example.git\" \\\napacheignite/ignite-docker ",
      "language": "shell"
    }
  ]
}
[/block]
 You should see the following print out in the logs:
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/3hddP5AaQTOpZosN00HN",
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

[block:api-header]
{
  "type": "basic",
  "title": "Start Bare Ignite Node"
}
[/block]
If `GIT_REPO` parameter is not configured, then Ignite docker container will download the specified or the latest Ignite distribution. By default the latest version is downloaded.

To pull the Ignite docker container use the following command:
[block:code]
{
  "codes": [
    {
      "code": "sudo docker pull apacheignite/ignite-docker",
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
    "0-0": "`IGNITE_URL`",
    "1-0": "`IGNITE_VERSION`",
    "2-0": "`IGNITE_SOURCE`",
    "0-1": "URL to the Ignite distribuition.",
    "1-1": "Ignite version.",
    "2-1": "Ignite edition which will be downloaded. This parameter is ignored, if IGNITE_VERSION is set.",
    "0-2": "`N/A`",
    "1-2": "`latest`",
    "2-2": "`COMMUNITY`",
    "0-3": "`http://apache-mirror.rbc.ru/pub/apache/incubator/ignite/1.1.0/`\n`apache-ignite-fabric-1.1.0-incubating-bin.zip`",
    "1-3": "`1.1.4`",
    "2-3": "`APACHE`",
    "3-0": "`IGNITE_CONFIG`",
    "3-1": "URL to the Ignite configuration file (can also be relative to the  META-INF folder on the class path). The downloaded config file will be saved to ./ignite-config.xml",
    "3-2": "`N/A`",
    "3-3": "`https://raw.githubusercontent.com/`\n`bob/master/ignite-cfg.xml`",
    "4-0": "`OPTION_LIBS`",
    "4-1": "Ignite optional libs which will be included in the class path.",
    "4-2": "`ignite-log4j,\nignite-spring,\nignite-indexing`",
    "4-3": "`ignite-aws,ignite-aop`"
  },
  "cols": 4,
  "rows": 5
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Example"
}
[/block]
To run Ignite docker container with bare Ignite node, use the following command:
[block:code]
{
  "codes": [
    {
      "code": "sudo docker run -it -e \"IGNITE_VERSION=1.1.0\" apacheignite/ignite-docker",
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
        "https://www.filepicker.io/api/file/lbgfn28TQKqOX03FTVsc",
        "docker_bare.png",
        "1113",
        "633",
        "#996220",
        ""
      ]
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Google Compute Deployment"
}
[/block]
1. To import the [Ignite Image](https://storage.googleapis.com/ignite-media/ignite-google-image-1.0.0.tar.gz), execute the following command:
[block:code]
{
  "codes": [
    {
      "code": "gcloud compute images \n  create <IMAGE_NAME> \n  --source-uri gs://ignite-media/ignite-google-image-1.0.0.tar.gz",
      "language": "shell"
    }
  ]
}
[/block]
For information please refer to [cloud.google.com](https://cloud.google.com/compute/docs/images#import_an_image)
    
2. Go to `Google Compute Console`.
3. Go to `Compute->Compute Engine->VM` instances and click on `New instance`.
4. Click on `Change` button in section `Boot disk`.
5. Go to `Your image` and choose imported image. On the screenshot the image is with `ignite-name` name.
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/uWZ9KlAKQeKzcaP9f7Sv",
        "Choose_image.png",
        "760",
        "625",
        "#4570b9",
        ""
      ]
    }
  ]
}
[/block]
6. Click on `Management, disk, networking, access & security options` and add any of the configuration parameters for the Ignite docker container.
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/dczZ8N7ATLCOV5X3KvKH",
        "Metadata.png",
        "683",
        "563",
        "#42609c",
        ""
      ]
    }
  ]
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
    "h-0": "Region",
    "0-0": "`US-WEST`",
    "1-0": "`US-EAST`",
    "2-0": "`EU-CENTRAL`",
    "0-1": "[ami-5b53a41f](https://console.aws.amazon.com/ec2/home?region=us-west-1#launchAmi=ami-5b53a41f)",
    "1-1": "[ami-3fa45e54](https://console.aws.amazon.com/ec2/home?region=us-east-1#launchAmi=ami-3fa45e54)",
    "2-1": "[ami-9c5f6481](https://console.aws.amazon.com/ec2/home?region=eu-central-1#launchAmi=ami-9c5f6481)",
    "h-1": "Image"
  },
  "cols": 2,
  "rows": 3
}
[/block]
2. Choose an `Instance Type`.
3. Go to `Configure Instance` and expand `Advanced Details` section.
4. Add any of the configuration parameters for the Ignite docker container.
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/tdxlrfjQqM4MSJpY8L6g",
        "AmazonUserData.png",
        "1112",
        "513",
        "#32465b",
        ""
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