* [Overview](#overview)
* [Kubernetes IP Finder](#kubernetes-ip-finder)
* [Kubernetes Ignite Lookup Service](#kubernetes-ignite-lookup-service)
* [Sharing Ignite Cluster Configuration](#sharing-ignite-cluster-configuration)
* [Ignite Pods Deployment](#ignite-pods-deployment)
* [Adjusting Ignite Cluster Size](#adjusting-ignite-cluster-size)
f
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Apache Ignite cluster can be easily deployed in and maintained by [Kubernetes](https://kubernetes.io) which is an open-source system for automating deployment, scaling, and management of containerized applications.

This getting started guide walks you through how to deploy an Apache Ignite cluster in Kubernetes environment.
[block:api-header]
{
  "type": "basic",
  "title": "Kubernetes IP Finder"
}
[/block]
To enable Apache Ignite nodes auto-discovery in Kubernetes, you need to enable `TcpDiscoveryKubernetesIpFinder` in `IgniteConfiguration`. Let's create an example configuration file called `example-kube.xml` and define the IP finder configuration as follows:
[block:code]
{
  "codes": [
    {
      "code": "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<beans xmlns=\"http://www.springframework.org/schema/beans\"\n       xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\"\n       xmlns:util=\"http://www.springframework.org/schema/util\"\n       xsi:schemaLocation=\"\n        http://www.springframework.org/schema/beans\n        http://www.springframework.org/schema/beans/spring-beans.xsd\n        http://www.springframework.org/schema/util\n        http://www.springframework.org/schema/util/spring-util.xsd\">\n\n<bean id=\"ignite.cfg\"\n    class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n\n    <property name=\"discoverySpi\">\n        <bean class=\"org.apache.ignite.spi.discovery.tcp.TcpDiscoverySpi\">\n        \t\t<property name=\"ipFinder\">\n            \t\t<bean\nclass=\"org.apache.ignite.spi.discovery.tcp.ipfinder.kubernetes.TcpDiscoveryKubernetesIpFinder\">\n            \t\t</bean>\n            </property>\n        </bean>\n    </property>\n</bean>\n</beans>\n",
      "language": "xml",
      "name": "example-kube.xml"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "Kubernetes IP finder",
  "body": "To learn more about Kubernetes IP finder and Apache Ignite nodes auto-discovery in Kuberentes environment, refer to [this documentation page](https://apacheignite-mix.readme.io/docs/kubernetes-discovery)."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Kubernetes Ignite Lookup Service"
}
[/block]
Ignite's `KubernetesIPFinder` requires users to configure and deploy a special Kubernetes service that maintains a list of the IP addresses of all the alive Ignite pods (nodes). 

Every time a new Ignite pod is started, the IP finder will connect to the service via the Kubernetes API to obtain the list of the existing Ignite pods' addresses. Using these addresses, the new node will be able to discover the rest of the cluster nodes and finally join the Apache Ignite cluster.

Let's configure the service the following way:
[block:code]
{
  "codes": [
    {
      "code": "apiVersion: v1\nkind: Service\nmetadata:\n  # Name of Ignite Service used by Kubernetes IP finder. \n  # The name must be equal to TcpDiscoveryKubernetesIpFinder.serviceName.\n  name: ignite\nspec:\n  clusterIP: None # custom value.\n  ports:\n    - port: 9042 # custom value.\n  selector:\n    # Must be equal to one of the labels set in Ignite pods'\n    # deployement configuration.\n    app: ignite",
      "language": "yaml",
      "name": "ignite-service.yaml"
    }
  ]
}
[/block]
and deploy it in Kubernetes using the command below:
[block:code]
{
  "codes": [
    {
      "code": "kubectl create -f ignite-service.yaml",
      "language": "shell"
    }
  ]
}
[/block]
Make sure the service is up and running:
[block:code]
{
  "codes": [
    {
      "code": " kubectl get svc ignite",
      "language": "shell"
    }
  ]
}
[/block]
The output should be similar to the one below:
[block:code]
{
  "codes": [
    {
      "code": "NAME      CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE\nignite    None         <none>        9042/TCP   29s",
      "language": "shell"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Sharing Ignite Cluster Configuration"
}
[/block]
Before you start deploying Ignite pods in Kubernetes using Apache Ignite [docker image](https://apacheignite.readme.io/docs/docker-deployment), you need to find a way on how to pass `example-kube.xml` (prepared above) to that docker image.

There are several approaches you can use. Here, we show you how to share the Ignite cluster configuration via a shared Kubernetes `PersistentVolume`.

Let's suppose you have some shared directory named `/data/ignite` that can be accessed by any Ignite pod. Go to this directory and copy `example-kube.xml` there.

Create a `PersistentVolume` configuration that will be backed by your real storage and will refer to the `/data/ignite` directory.
[block:code]
{
  "codes": [
    {
      "code": "kind: PersistentVolume\napiVersion: v1\nmetadata:\n  name: ignite-volume\n  labels:\n    type: local\nspec:\n  capacity:\n    storage: 1Gi\n  accessModes:\n    - ReadWriteOnce\n  hostPath:\n    path: \"/data/ignite\"",
      "language": "yaml",
      "name": "ignite-volume.xml"
    }
  ]
}
[/block]
 Deploy the volume using the command below:
[block:code]
{
  "codes": [
    {
      "code": "kubectl create -f ignite-volume.yaml",
      "language": "shell"
    }
  ]
}
[/block]
Check that the volume was deployed and available for usage:
[block:code]
{
  "codes": [
    {
      "code": "kubectl get pv ignite-volume",
      "language": "shell"
    }
  ]
}
[/block]
The output should be similar to the one below:
[block:code]
{
  "codes": [
    {
      "code": "NAME            CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     REASON    AGE\nignite-volume   1Gi        RWO           Retain          Available                       3m",
      "language": "shell"
    }
  ]
}
[/block]
Finally, create a `PersistentVolumeClaim` that will be automatically bound to the `PersistentVolume` initiated above:
[block:code]
{
  "codes": [
    {
      "code": "kind: PersistentVolumeClaim\napiVersion: v1\nmetadata:\n  name: ignite-volume-claim\nspec:\n  accessModes:\n    - ReadWriteOnce\n  resources:\n    requests:\n      storage: 1Gi",
      "language": "yaml",
      "name": "ignite-volume-claim.yaml"
    }
  ]
}
[/block]
Create the PersistentVolumeClaim using the configuration above:
[block:code]
{
  "codes": [
    {
      "code": "kubectl create -f ignite-volume-claim.yaml ",
      "language": "shell"
    }
  ]
}
[/block]
Make sure that the `PersistentVolumeClaim` gets created and was bound to the `PersistentVolume` that stores `example-kube.xml` configuration.
[block:code]
{
  "codes": [
    {
      "code": "kubectl get pvc ignite-volume-claim\n\nNAME                  STATUS    VOLUME          CAPACITY   ACCESSMODES   AGE\nignite-volume-claim   Bound     ignite-volume   1Gi        RWO           2m\n\nkubectl get pv ignite-volume\n\nNAME            CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                         REASON    AGE\nignite-volume   1Gi        RWO           Retain          Bound     default/ignite-volume-claim             13m",
      "language": "shell"
    }
  ]
}
[/block]
Now, it's time to prepare a Kubernetes deployment configuration for Ignite pods and deploy them
[block:api-header]
{
  "type": "basic",
  "title": "Ignite Pods Deployment"
}
[/block]
Finally, let's define a YAML configuration for Ignite pods:
[block:code]
{
  "codes": [
    {
      "code": "# An example of a Kubernetes configuration for Ignite pods deployment.\napiVersion: extensions/v1beta1\nkind: Deployment\nmetadata:\n  # Custom Ignite cluster's name.\n  name: ignite-cluster\nspec:\n  # A number of Ignite pods to be started by Kubernetes initially.\n  replicas: 2\n  template:\n    metadata:\n      labels:\n        # This label has to be added to the selector's section of \n        # ignite-service.yaml so that the Kubernetes Ignite lookup service\n        # can easily track all Ignite pods available deployed so far.\n        app: ignite\n    spec:\n      volumes:\n        # Custom name for the storage that holds Ignite's configuration\n        # which is example-kube.xml.\n        - name: ignite-storage\n          persistentVolumeClaim:\n           # Must be equal to the PersistentVolumeClaim created before.\n           claimName: ignite-volume-claim\t\n      containers:\n        # Custom Ignite pod name.\n      - name: ignite-node\n        # Ignite Docker image. Kubernetes IP finder is supported starting from\n        # Apache Ignite 1.9.\n        image: apacheignite/ignite:1.9\n        env:\n        # Ignite's Docker image parameter. Adding the jar file that\n        # contain TcpDiscoveryKubernetesIpFinder implementation.\n        - name: OPTION_LIBS\n          value: ignite-kubernetes\n        # Ignite's Docker image parameter. Passing the Ignite configuration\n        # to use for an Ignite pod.\n        - name: CONFIG_URI\n          value: file:////data/ignite/example-kube.xml\n        ports:\n        # Ports to open.\n        # Might be optional depending on your Kubernetes environment.\n        - containerPort: 11211 # REST port number.\n        - containerPort: 47100 # communication SPI port number.\n        - containerPort: 47500 # discovery SPI port number.\n        - containerPort: 49112 # JMX port number.\n        volumeMounts:\n        # Mounting the storage with the Ignite configuration.\n        - mountPath: \"/data/ignite\"\n          name: ignite-storage\n          \n",
      "language": "yaml",
      "name": "ignite-deployment.xml"
    }
  ]
}
[/block]
As you can see, the configuration defines a couple of environment variables (`OPTION_LIBS` and `CONFIG_URIL`) that will be processed by a special shell script used by Ignite's docker image. The full list of docker image's configuration parameters is available on [Docker Deployment](doc:docker-deployment) page.

Next, go ahead and deploy Ignite pods in Kubernetes using the configuration​ above:
[block:code]
{
  "codes": [
    {
      "code": "kubectl create -f ignite-deployment.yaml",
      "language": "shell"
    }
  ]
}
[/block]
Check that Ignite pods are up and running:
[block:code]
{
  "codes": [
    {
      "code": "kubectl get pods",
      "language": "shell"
    }
  ]
}
[/block]
Pick a name of one of the pods available 
[block:code]
{
  "codes": [
    {
      "code": "NAME                              READY     STATUS    RESTARTS   AGE\nignite-cluster-3454482164-d4m6g   1/1       Running   0          25m\nignite-cluster-3454482164-w0xtx   1/1       Running   0          25m",
      "language": "shell"
    }
  ]
}
[/block]
and get the logs from it making sure that both Ignite pods were able to discover each other and from the cluster:
[block:code]
{
  "codes": [
    {
      "code": "kubectl logs ignite-cluster-3454482164-d4m6g",
      "language": "shell"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Adjusting Ignite Cluster Size"
}
[/block]
You can adjust Apache Ignite cluster size on the fly using the standard Kubernetes API. For instance, if you want to scale out the cluster from 2 to 5 nodes then the command below can be used:
[block:code]
{
  "codes": [
    {
      "code": "kubectl scale --replicas=5 -f ignite-deployment.yaml",
      "language": "shell"
    }
  ]
}
[/block]
Double check the cluster was scaled out successfully:
[block:code]
{
  "codes": [
    {
      "code": "kubectl get pods",
      "language": "shell"
    }
  ]
}
[/block]
The output should show that you now have 5 Ignite pods up and running:
[block:code]
{
  "codes": [
    {
      "code": "NAME                              READY     STATUS    RESTARTS   AGE\nignite-cluster-3454482164-d4m6g   1/1       Running   0          34m\nignite-cluster-3454482164-ktkrr   1/1       Running   0          58s\nignite-cluster-3454482164-r20f8   1/1       Running   0          58s\nignite-cluster-3454482164-vf8kh   1/1       Running   0          58s\nignite-cluster-3454482164-w0xtx   1/1       Running   0          34m",
      "language": "text"
    }
  ]
}
[/block]