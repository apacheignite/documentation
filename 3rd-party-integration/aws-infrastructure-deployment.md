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
As far as framework uses Amazon cloud infrastructure it requires some prerequisites to be setup in the Amazon account where you are going to deploy all the staff. Here is the list of prerequisites:

1. [AWS account](http://docs.aws.amazon.com/lambda/latest/dg/setup.html) from which you are going to deploy all the `Cassandra/Ignite/Tests` clusters infrastructure should have permissions to create [EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Instances.html) instances.
2. The framework uses [S3](https://aws.amazon.com/s3/) to coordinate activities between `Tests` nodes and store some metadata about the system. Thus you should have some folder on S3 which is dedicated as a root folder to store all this staff.
3. There should be [IAM Role](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) which grants full access (READ/WRITE/DELETE) to the folder specified above.
4. The IAM Role specified above, should have [permissions](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_permissions.html) to list IAM roles and assign the role specified above to EC2 instances.
5. The IAM Role specified above, should have [permissions](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_permissions.html) to tag EC2 instances. This requirement is optional and is only needed if you want to [tag](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html) your EC2 instances. It actually makes sense when you are working with rather big EC2 environments to being able to distinguish different EC2 instances.
6. The AWS account you are using, should have access to select from existing or create new [security group](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)
[block:api-header]
{
  "type": "basic",
  "title": "Building framework"
}
[/block]
Framework itself is represented as a set of bootstrap shell scripts, which can be used from [EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html) console to launch appropriate number of `Ignite/Cassandra/Tests` EC2 instances and provided as a part of tests source code of the module. Thus first of all you should [build Ignite distribution](https://apacheignite.readme.io/docs/getting-started#section-building-from-source) from the source code.

After building Ignite distribution from the source code you will be able to find `target/tests-package` directory and its zip archive `target/ignite-cassandra-tests-<version>.zip` (**tests package** further) inside Cassandra module directory. The structure of the test package is explained in [Load tests](https://github.com/irudyak/ignite/wiki/Load-tests#building-load-tests) document.

All the details of the Amazon infrastructure deployment are configured by scripts from `bootstrap/aws` directory of the tests package. 

Now lets take a look at the `bootstrap` directory and framework configuration settings in more details.
[block:api-header]
{
  "type": "basic",
  "title": "Framework structure"
}
[/block]
All infrastructure deployment settings are specified in the shell scripts inside `bootstrap` directory of the tests package. The directory itself has such a structure:
* **aws** - root directory for AWS bootstrap shell scripts.
  * **cassandra** - root directory for shell scripts to spin up Cassandra cluster.
     * **cassandra-bootstrap.sh** - bootstrap script for EC2 nodes of Cassandra cluster. You should only modify **TESTS_PACKAGE_DONLOAD_URL** environment variable (see next chapters).
     * **cassandra-env.sh** - Cassandra daemon environment configuration script. You can modify it according to your custom use-case, but be careful cause incorrect modifications could prevent Cassandra daemon to start correctly.
	 * **cassandra-start.sh** - shell script used by the framework to start Cassandra cluster. **Don't modify this file**.
	 * **cassandra-template.yaml** - Cassandra YAML configuration file. You can modify it according to your custom use-case, but be careful cause incorrect modifications could prevent Cassandra daemon to start correctly.
  * **ganglia** - root directory for shell scripts to spin up [Ganglia](http://ganglia.info/) monitoring system.
	 * **agent-bootstrap.sh** - Ganglia agent bootstrap script. **Don't modify this file**.
	 * **agent-start.sh** - Ganglia agent start script. **Don't modify this file**.
	 * **master-bootstrap.sh** - Ganglia master bootstrap script. You should only modify **TESTS_PACKAGE_DONLOAD_URL** environment variable (see next chapters). 
  * **ignite** - root directory for shell scripts to spin up Ignite cluster.
	 * **ignite-bootstrap.sh** - bootstrap script for EC2 nodes of Ignite cluster. You should only modify **TESTS_PACKAGE_DONLOAD_URL** environment variable (see next chapters).
	 * **ignite-cassandra-server-template.xml** - Ignite daemon Spring context configuration. You can modify it according to your custom use-case, but be careful cause incorrect modifications could prevent Ignite daemon to start correctly.
	 * **ignite-env.sh** - Ignite daemon environment configuration script. You can modify it according to your custom use-case, but be careful cause incorrect modifications could prevent Ignite daemon to start correctly. 
	 * **ignite-start.sh** - shell script used by the framework to start Ignite cluster. **Don't modify this file**.   
  * **tests** - root directory for shell scripts to spin up Tests cluster.
	 * **ignite-cassandra-client-template.xml** - Ignite client Spring context configuration. You can modify it according to your custom use-case, but be careful cause incorrect modifications could prevent Ignite client to connect to server.
	 * **tests-bootstrap.sh** - bootstrap script for EC2 nodes of Tests cluster. You should only modify **TESTS_PACKAGE_DONLOAD_URL** environment variable (see next chapters).
	 * **tests-manager.sh** - tests manager daemon script. **Don't modify this file**.
	 * **tests-report.sh** - shell script which is responsible for collecting tests statistic from all EC2 instances, generating summary report and uploading it to S3. **Don't modify this file**.     
  * **common.sh** - shell script defining common functions to reuse. **Don't modify this file**.
  * **env.sh** - shell scripts defining environment variables for the infrastructure. This is the main script to specify settings for AWS infrastructure which should be created (see next chapters).
  * **logs-collector.sh** - shell script for logs collector daemon, which is responsible for collecting logs from all EC2 instances and uploading them to S3. **Don't modify this file**.