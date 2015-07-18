Docker allows to package Ignite deployment with all the dependencies into a standardized container. Docker automates downloading the Ignite release, deploying users' code into Ignite, and configuring nodes. It also automatically starts up a fully configured Ignite node.

Ignite docker container can run it two modes:  
[block:api-header]
{
  "type": "basic",
  "title": "Start from User Git Repository"
}
[/block]
The container is in this mode, if `GIT_REPO` parameter is configured. Docker container is building user's project which downloaded from git repository and will run the version of the Ignite that using in given repository. Artifacts which have built will be avalible for Ignite node. 

For running pulling docker container use the following command:

   `sudo docker pull apacheignite/ignite-docker`

All configuration is handled through environment variables in docker container. Following configuration parameters can be configured:
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
    "0-1": "The url to git repository which will be built.",
    "1-1": "The branch which which will be built.",
    "2-1": "The command which will be building project.",
    "0-2": "`false`",
    "1-2": "`true`",
    "2-2": "`true`",
    "0-3": "`N/A`",
    "1-3": "`master`",
    "2-3": "`mvn clean package`",
    "0-4": "`https://github.com/bob/ignite-pojo`",
    "1-4": "`sprint-1`",
    "2-4": "`mvn clean package -DskipTests=true`",
    "3-2": "`true`",
    "3-0": "`IGNITE_CONFIG`",
    "3-1": "The url to config file or relativity path by META-INF. Downloaded config file will be saved to ./ignite-config.xml",
    "3-3": "`N/A`",
    "3-4": "`https://raw.githubusercontent.com/bob/master/ignite-cfg.xml`",
    "4-0": "`LIB_PATTERN`",
    "4-1": "If it's set then will copy only files which match by pattern.",
    "4-2": "`true`",
    "4-3": "`copy all jar files from target folder`",
    "5-0": "`OPTION_LIBS`",
    "5-1": "Ignite optional libs which will be include in classpath.",
    "5-2": "`true`",
    "5-3": "`ignite-log4j,ignite-spring,ignite-indexing`",
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
For running from git repository execute the following:

  `sudo docker run -it -e "GIT_REPO=https://github.com/TikhonovNikolay/docker-example.git" apacheignite/ignite-docker`

Should see the following in logs:
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
If `GIT_REPO` parameter is not configured then Docker container will download an Ignite distributive. By default container is downloading latest version because it contains the last feature and bug fixes. 

For running pulling docker container use the following command:

   `sudo docker pull apacheignite/ignite-docker`

All configuration is handled through environment variables in docker container. Following configuration parameters can be optionally configured:
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
    "0-1": "The url to Ignite distributive which will be run.",
    "1-1": "The version ignite which will be run on nodes.",
    "2-1": "The ignite edition which will be downloaded. This parameter ignored, if IGNITE_VERSION is set.",
    "0-2": "`N/A`",
    "1-2": "`latest`",
    "2-2": "`COMMUNITY`",
    "0-3": "`http://apache-mirror.rbc.ru/pub/apache/incubator/ignite/1.1.0/\napache-ignite-fabric-1.1.0-incubating-bin.zip`",
    "1-3": "`1.1.4`",
    "2-3": "`APACHE`",
    "3-0": "`IGNITE_CONFIG`",
    "3-1": "The url to config file or relativity path by META-INF. Downloaded config file will be saved to ./ignite-config.xml",
    "3-2": "`N/A`",
    "3-3": "`https://raw.githubusercontent.com/bob/master/ignite-cfg.xml\n`",
    "4-0": "`OPTION_LIBS`",
    "4-1": "Ignite optional libs which will be include in classpath.",
    "4-2": "`ignite-log4j,ignite-spring,ignite-indexing`",
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
For running from git repository execute the following:

  `sudo docker run -it -e "IGNITE_VERSION=1.1.0" apacheignite/ignite-docker`

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
1. Import [Ignite Image](https://storage.googleapis.com/ignite-media/ignite-google-image-1.0.0.tar.gz), for this run the following command:

    `gcloud compute images create <IMAGE_NAME> --source-uri gs://ignite-media/ignite-google-image-1.0.0.tar.gz`
     For information please refer to [cloud.google.com](https://cloud.google.com/compute/docs/images#import_an_image)
    
2. Go to `google compute console`.
3. Go to `Compute->Compute Engine->VM` instances and click on `New instance`.
4. Click to `Change` button in section `Boot disk`.
5. Go to `Your image` and choose imported image. On screenshot is image with `ignite-name` name.
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
6. Click on `Management, disk, networking, access & security options` and add parameters for docker container which describe above.
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
7. Fill required fields and run instances.
8. Conect to instances.
9. For accessing to execution progress need to know a `container id`. The following command will show it.

    `sudo docker ps`
    
10. Show logs.

    `sudo docker logs -f CONTAINER_ID`
    
11. Enter to docker container.

    `sudo docker exec -it container_id /bin/bash`
[block:api-header]
{
  "type": "basic",
  "title": "Amazon EC2 Deployment"
}
[/block]
1. Choose required region and click on link in table below.
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
4. Configure parameters for docker container which describe above.
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
5. On the Tag Instance set value for Name tag. For example `ignite-node`
6. Review and run instances.
7. Conect to instances [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AccessingInstances.html)
8. For accessing to execution progress need to know a `container id`. The following command will show it.

    `sudo docker ps`
    
9. Show logs.

    `sudo docker logs -f CONTAINER_ID`
    
10. Enter to docker container.

    `sudo docker exec -it container_id /bin/bash`