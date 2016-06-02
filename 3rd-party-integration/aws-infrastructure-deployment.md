AWS infrastructure deployment framework simplifies deployment of `Cassandra/Ignite/Tests` clusters in [Amazon](https://aws.amazon.com/products/?nc2=h_ql_sf_ls) and automates all the routine related to:

1. Creation and bootstrapping of required amount of EC2 instances for `Ignite/Cassandra/Tests` clusters
2. Run load tests and wait for their completion
3. Collect all the load tests statistics from each EC2 instance and produce summary report
4. Provides [Ganglia](http://ganglia.info/) monitoring for your `Ignite/Cassandra/Tests` clusters, thus you can see how your clusters look like under high load  :-)
[block:api-header]
{
  "type": "basic",
  "title": "Concepts"
}
[/block]
Lets now dive deeper in the details and the whole idea of the framework step by step:

1. At the high level, you can just think about the framework as a set of bootstrap shell scripts to create  `Ignite/Cassandra/Tests` clusters and `Ganglia` monitoring.
2. To spin up `Ignite/Cassandra/Tests` cluster or `Ganglia` you can just use [EC2 instance launch wizard](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/launching-instance.html) or [Spot instances launch wizard](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-requests.html) where you should specify appropriate number of instances for your cluster and bootstrap script in the `User Data` section. As another option you can just use AWS CLI where you can specify number of EC2 instances and bootstrap script as `User Data` section.
3. Thus you just need to repeat the previous step for each type of the cluster `Ignite/Cassandra/Tests` and optionally for `Ganglia` (if you want to monitor your infrastructure). 

The cool feature is that you don't need to wait for one cluster up and running to start creating another cluster. For example `Ignite` cluster nodes need information about `Cassandra` cluster nodes to correctly setup persistent store settings, `Tests` cluster nodes need info about nodes of both `Ignite` & `Cassandra` clusters and etc. Using the framework you can just launch creation process for all types of the clusters in parallel and framework will take care about all such details. Moreover it's the preferred way to deploy your infrastructure, cause it takes the minimum time to setup everything.