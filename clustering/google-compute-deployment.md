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
6. Click on `Management, disk, networking, access & security options` and add any of the configuration parameters for the Ignite docker container.
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