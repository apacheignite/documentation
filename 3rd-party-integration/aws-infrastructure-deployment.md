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
[block:callout]
{
  "type": "success",
  "title": "That's all you need to do to setup your infrastructure."
}
[/block]
The cool feature is that you don't need to wait for one cluster up and running to start creating another cluster. For example `Ignite` cluster nodes need information about `Cassandra` cluster nodes to correctly setup persistent store settings, `Tests` cluster nodes need info about nodes of both `Ignite` & `Cassandra` clusters and etc. Using the framework you can just launch creation process for all types of the clusters in parallel and framework will take care about all such details. Moreover it's the preferred way to deploy your infrastructure, cause it takes the minimum time to setup everything.


Framework will take care about all such details like:
  * Correctly spinning up all EC2 nodes of Cassandra cluster thus that:
     * All of them will be in the same cluster
  * Correctly spinning up all EC2 nodes of Ignite cluster thus that:
     * All of them will be in the same cluster
     * All of them will reference appropriate Cassandra cluster seeds in persistent store configuration settings
  * Correctly spinning up all EC2 nodes of Tests cluster thus that:
     * All of them will be in the same cluster
     * All of them will reference appropriate Cassandra cluster seeds in load tests settings
     * All of them will reference appropriate Ignite cluster nodes in load tests settings
	 * Automatically launch load tests on all nodes only when:
	    * All `Cassandra` nodes up and running
	    * All `Ignite` nodes up and running
	    * All `Tests` nodes up and running and ready to launch tests
  * Collect load tests statistic from all EC2 nodes of `Tests` cluster, produces summary report and upload it to S3
[block:api-header]
{
  "type": "basic",
  "title": "Prerequisites"
}
[/block]
As far as framework using Amazon infrastructure it requires some prerequisites to be setup in the Amazon account where you are going to deploy all the staff. Here is the list of prerequisites:

1. [AWS account](http://docs.aws.amazon.com/lambda/latest/dg/setup.html) from which you are going to deploy all the `Cassandra/Ignite/Tests` clusters infrastructure should have permissions to create [EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Instances.html) instances.
2. The framework uses [S3](https://aws.amazon.com/s3/) to coordinate activities between `Tests` nodes and store some metadata about the system. Thus you should have some folder on S3 which is dedicated as a root folder to store all this staff.
3. There should be [IAM Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) which grants full access (READ/WRITE/DELETE) to the folder specified above.
4. The IAM Role specified above, should have [permissions](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_permissions.html) to list IAM roles and assign the role specified above to EC2 instances.
5. The IAM Role specified above, should have [permissions](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_permissions.html) to tag EC2 instances. This requirement is optional and is only needed if you want to [tag](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html) your EC2 instances. It actually makes sense when you are working with rather big EC2 environments to being able to distinguish different EC2 instances.
6. The AWS account you are using, should have access to select from existing or create new [security group](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)