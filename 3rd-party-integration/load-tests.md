Ignite Cassandra module provides a set of load tests which allows you to simulate production load for Ignite and Cassandra with your custom key/value classes and persistence settings. Thus using these load tests you can measure performance characteristics of the system for your specific configuration:
* Set of custom key/value classes
* Set of Ignite nodes
* Set of Cassandra nodes

Such kind of measurements could be useful for better understanding of system scalability and sizing.
[block:api-header]
{
  "type": "basic",
  "title": "Building Load Tests"
}
[/block]
Load tests for Cassandra module are provided as a part of tests source code of the module. Thus first of all you should [build Ignite distribution](doc:getting-started#section-building-from-source) from the source code. 

After building Ignite distribution from the source code you will be able to find `target/tests-package` directory inside Cassandra module directory and `target/ignite-cassandra-tests-<version>.zip` zip archive of this directory (**tests package**). Tests package contains ready to use load tests application for Ignite Cassandra module and has such structure:

* **bootstrap** - directory containing bootstrap scripts for [AWS infrastructure deployment](doc:aws-infrastructure-deployment) framework

* **lib** - directory containing all the required **jars** to communicate with Ignite and Cassandra. If you are going to run load tests for your custom key/value classes, you should create a separate **jar** for them and put it inside this directory (see more details in the next chapters).

* **settings** - directory containing load tests configuration settings like Cassandra and Ignite connection details, persistent settings and so on (see more details in the next chapters).

* **cassandra-load-tests.bat** - shell script for MS Windows, to run persistence load tests directly against Cassandra cluster. This test allows you to measure the performance of direct key/value persistence operations for your Cassandra cluster bypassing Ignite cluster. This could be rather useful, cause based on the test results you'll be able to select Cassandra cluster of appropriate capacity to be used as a persistence store for your Ignite cache.

* **cassandra-load-tests.sh** - the same shell script like above, but for Linux. 

* **ignite-load-tests.bat** - shell script for MS Windows, to run persistence load tests against Ignite cluster. This test allows you to measure the performance of key/value persistence operations for your Ignite cluster with a Cassandra cluster configured to be used as a persistence store. Based on the test results you'll be able to select Ignite cluster of appropriate capacity.

* **ignite-load-tests.sh** - the same shell script like above, but for Linux.

* **jvm-opts.bat** - shell script for MS Windows, to specify JVM settings for load tests.

* **jvm-opts.sh** - the same shell script like above, but for Linux.

* **recreate-cassandra-artifacts.bat** - shell script to recreate Cassandra artifacts (keyspaces and tables) before running load tests (used by AWS load tests infrastructure deployment scripts, thus you don't need to run it manually).

* **recreate-cassandra-artifacts.sh** - the same shell script like above, but for Linux.

As you just seen (from the list of shell scripts) there are actually two types of load tests:
* **Cassandra load tests** - performs all key/value persistence operations directly with Cassandra bypassing Ignite.

* **Ignite load tests** - performs all key/value persistence operations with Ignite using appropriate instance of [IgniteCache](doc:getting-started#first-ignite-data-grid-application).

Now lets take a look at load tests scenarios and configuration details.
[block:api-header]
{
  "type": "basic",
  "title": "Load tests scenarios"
}
[/block]
Both `Cassandra` and `Ignite` load tests scripts are using exactly the same set of tests:

1. Load test for single **WRITE** operation. Such an operation is performed when you calling `IgniteCache.put` method of your cache.
2. Load test for **BULK_WRITE** operation. Such an operation is performed when you calling `IgniteCache.putAll` method of your cache.
3. Load test for signle **READ** operation. Such an operation is performed when you calling `IgniteCache.get` method of your cache.
4. Load test for **BULK_READ** operation. Such an operation is performed when you calling `IgniteCache.getAll` method of your cache.

All of the specified load tests will be executed sequentially one after another in the provided order.
[block:api-header]
{
  "type": "basic",
  "title": "Load Tests Settings"
}
[/block]
Load test settings are specified inside these property files from `settings` folder:

1. **log4j.properties** - specifies logger settings for load tests.
2. **org/apache/ignite/tests/cassandra/connection.properties** - specifies contact points to use for Cassandra connection (see details below).
3. **org/apache/ignite/tests/cassandra/credentials.properties** - specifies credentials to use for Cassandra connection (see details below).
4. **tests.properties** - specifies load tests execution settings (see details below).

While `log4j.properties` file is rather simple and straightforward configuration of `log4j` logger lets look more deeply inside other configuration files:
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">org/apache/ignite/tests/cassandra/connection.properties</div>"
}
[/block]
| Property      | Description |
| :-------------| :-----|
| <sup>**contact.points**      | <sup>Comma separated list of Cassandra nodes to use as contact points. Should be is such a format: **server-1[:port], server-2[:port], server-3[:port]** and etc.|
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">org/apache/ignite/tests/cassandra/credentials.properties</div>"
}
[/block]
| Property      | Description |
| :-------------| :-----|
| <sup>**admin.user**      | <sup>Admin user name to connect to Cassandra|
| <sup>**admin.password**      | <sup>Admin user password to connect to Cassandra|
| <sup>**regular.user**      | <sup>Regular user name to connect to Cassandra|
| <sup>**regular.password**      | <sup>Regular user password to connect to Cassandra|
[block:html]
{
  "html": "<div style=\"color:green;font-weight: bold;font-size: 120%;\">tests.properties</div>"
}
[/block]
| Property      | Description |
| :-------------| :-----|
| <sup>**bulk.operation.size**      | <sup>Number of elements used for each bulk operation attempt: `IgniteCache.getAll`, `IgniteCache.putAll` |
| <sup>**load.tests.cache.name**  |   <sup>Ignite cache to be used by load tests |
| <sup>**load.tests.threads.count**  |   <sup>Number of threads used for each load test |
| <sup>**load.tests.warmup.period**  |   <sup>Warm up period (in milliseconds) for each load test before starting any measurements |
| <sup>**load.tests.execution.time**  |   <sup>Time for each load test execution (in milliseconds), excluding warm up period|
| <sup>**load.tests.requests.latency**  |   <sup>Latency (in milliseconds) between two sequential requests to Cassandra/Ignite |
| <sup>**load.tests.persistence.settings**  |   <sup>Resource specifying Cassandra persistence settings for load tests |
| <sup>**load.tests.ignite.config**  |   <sup>Resource specifying Ignite cluster connection settings for load tests |
| <sup>**load.tests.key.generator**  |   <sup>Key object generator for Ignite cache (see below) |
| <sup>**load.tests.value.generator**  |   <sup>Value object generator for Ignite cache (see below) |
[block:api-header]
{
  "type": "basic",
  "title": "Running Load Tests"
}
[/block]
Before running load tests make sure that:

1. All nodes of your Ignite cluster are configured to use the same Cassandra cache store configuration settings. You can find an example of remote Ignite node configuration in a resource file `settings/org/apache/ignite/tests/persistence/primitive/ignite-remote-server-config.xml` from tests source code. If you plan to use only Cassandra load tests you don't need this step.

2. `load.tests.ignite.config` property of `tests.properties` points to correct Ignite client node configuration (which should have exactly the same Cassandra connection and persistence settings like remote Ignite node config file). If you plan to use only Cassandra load tests you don't need this step.

3. Cassandra connection settings specified correctly in `org/apache/ignite/tests/cassandra/connection.properties` and `org/apache/ignite/tests/cassandra/credentials.properties` files.

After that you can easily run load tests just executing appropriate shell script:
* **cassandra-load-tests.sh / cassandra-load-tests.bat** - shell scripts to run persistence load tests directly against Cassandra cluster. This test allows you to measure the performance of direct key/value persistence operations for your Cassandra cluster bypassing Ignite cluster. This could be rather useful, cause based on the test results you'll be able to select Cassandra cluster of appropriate capacity to be used as a persistence store for your Ignite cache.

* **ignite-load-tests.sh / ignite-load-tests.bat** - shell scripts to run persistence load tests against Ignite cluster. This test allows you to measure the performance of key/value persistence operations for your Ignite cluster with a Cassandra cluster configured to be used as a persistence store. Based on the test results you'll be able to select Ignite cluster of appropriate capacity.
[block:api-header]
{
  "type": "basic",
  "title": "Using Custom Key/Value Classes"
}
[/block]
If you want to use your custom key/value classes for load tests you should:

1. Specify generator for your custom key class in `load.tests.key.generator` property of `tests.properties`. The idea of **generator** class is pretty simple - it's responsible for generating instances of particular class and should implement such method `public Object generate(long i)` of `org.apache.ignite.tests.load.Generator` interface. You can also find such implementation examples:
   * **org.apache.ignite.tests.load.IntGenerator** - generates `int` instances
   * **org.apache.ignite.tests.load.LongGenerator** - generates `long` instances
   * **org.apache.ignite.tests.load.PersonIdGenerator** - generates instances of custom Ignite cache key class `org.apache.ignite.tests.pojos.PersonId`
   * **org.apache.ignite.tests.load.PersonIdGenerator** - generates instances of custom Ignite cache value class `org.apache.ignite.tests.pojos.Person`

2. Specify generator for your custom value class in `load.tests.value.generator` property of `tests.properties`. Actually it's the same generator like mentioned above, but instead of key objects it should generate appropriate custom value objects.

3. Specify Cassandra persistence settings for you custom key/value classes, put persistence settings configuration file inside `settings` folder and check that `load.tests.persistence.settings` property of `tests.properties` points to your custom persistence settings configuration.

4. Define Ignite cache configuration so that it will use your custom persistence settings and check that `load.tests.ignite.config` property of `tests.properties` points to your Ignite cache configuration file.

5. Create **jar** file containing you custom key/value classes and corresponding **generator** classes for them and put it (and its 3rd party dependencies if any) inside `lib` directory.

That's all the steps you need to run load tests using your custom classes.
[block:api-header]
{
  "type": "basic",
  "title": "Analyzing Tests Execution Results"
}
[/block]
The results of load tests execution are provided as `log4j` log files. By default (if you didn't modify `log4j.properties` file) there are two files which shows summary results of tests execution:

1. **cassandra-load-tests.log** - contains summary statistic of Cassandra load tests execution results (if you run `cassandra-load-tests.sh` or `cassandra-load-tests.bat` shell script).
2. **ignite-load-tests.log** - contains summary statistic of Ignite load tests execution results (if you run `ignite-load-tests.sh` or `ignite-load-tests.bat` shell script).

Here is an example of `ignite-load-tests.log` (log for Cassandra load tests looks pretty much the same):
[block:code]
{
  "codes": [
    {
      "code": "19:53:37,303  INFO [main] - Ignite load tests execution started\n19:53:37,305  INFO [main] - Running WRITE test\n19:53:37,305  INFO [main] - Setting up load tests driver\n19:53:42,352  INFO [main] - Load tests driver setup successfully completed\n19:53:42,355  INFO [main] - Starting workers\n19:53:42,384  INFO [main] - Workers started\n19:53:42,385  INFO [main] - Waiting for workers to complete\n20:01:42,398  INFO [main] - Worker WRITE-worker-0 successfully completed\n20:01:42,407  INFO [main] - Worker WRITE-worker-1 successfully completed\n20:01:42,452  INFO [main] - Worker WRITE-worker-2 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-3 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-4 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-5 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-6 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-7 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-8 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-9 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-10 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-11 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-12 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-13 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-14 successfully completed\n20:01:42,453  INFO [main] - Worker WRITE-worker-15 successfully completed\n20:01:42,454  INFO [main] - Worker WRITE-worker-16 successfully completed\n20:01:42,454  INFO [main] - Worker WRITE-worker-17 successfully completed\n20:01:42,639  INFO [main] - Worker WRITE-worker-18 successfully completed\n20:01:42,639  INFO [main] - Worker WRITE-worker-19 successfully completed\n20:01:42,639  INFO [main] - WRITE test execution successfully completed.\n20:01:42,639  INFO [main] - \n-------------------------------------------------\nWRITE test statistics\nWRITE messages: 1681780\nWRITE errors: 0, 0.00%\nWRITE speed: 5597 msg/sec\n-------------------------------------------------\n20:01:42,695  INFO [main] - Running BULK_WRITE test\n20:01:42,695  INFO [main] - Setting up load tests driver\n20:01:45,074  INFO [main] - Load tests driver setup successfully completed\n20:01:45,074  INFO [main] - Starting workers\n20:01:45,093  INFO [main] - Workers started\n20:01:45,094  INFO [main] - Waiting for workers to complete\n20:09:45,084  INFO [main] - Worker BULK_WRITE-worker-0 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-1 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-2 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-3 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-4 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-5 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-6 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-7 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-8 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-9 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-10 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-11 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-12 successfully completed\n20:09:45,105  INFO [main] - Worker BULK_WRITE-worker-13 successfully completed\n20:09:45,254  INFO [main] - Worker BULK_WRITE-worker-14 successfully completed\n20:09:45,254  INFO [main] - Worker BULK_WRITE-worker-15 successfully completed\n20:09:45,254  INFO [main] - Worker BULK_WRITE-worker-16 successfully completed\n20:09:45,258  INFO [main] - Worker BULK_WRITE-worker-17 successfully completed\n20:09:45,258  INFO [main] - Worker BULK_WRITE-worker-18 successfully completed\n20:09:45,258  INFO [main] - Worker BULK_WRITE-worker-19 successfully completed\n20:09:45,258  INFO [main] - BULK_WRITE test execution successfully completed.\n20:09:45,258  INFO [main] - \n-------------------------------------------------\nBULK_WRITE test statistics\nBULK_WRITE messages: 2021500\nBULK_WRITE errors: 0, 0.00%\nBULK_WRITE speed: 6748 msg/sec\n-------------------------------------------------\n20:09:45,477  INFO [main] - Running READ test\n20:09:45,477  INFO [main] - Setting up load tests driver\n20:09:48,626  INFO [main] - Load tests driver setup successfully completed\n20:09:48,626  INFO [main] - Starting workers\n20:09:48,631  INFO [main] - Workers started\n20:09:48,631  INFO [main] - Waiting for workers to complete\n20:17:57,128  INFO [main] - Worker READ-worker-0 successfully completed\n20:17:57,128  INFO [main] - Worker READ-worker-1 successfully completed\n20:17:57,128  INFO [main] - Worker READ-worker-2 successfully completed\n20:17:57,128  INFO [main] - Worker READ-worker-3 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-4 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-5 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-6 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-7 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-8 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-9 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-10 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-11 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-12 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-13 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-14 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-15 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-16 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-17 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-18 successfully completed\n20:17:57,145  INFO [main] - Worker READ-worker-19 successfully completed\n20:17:57,145  INFO [main] - READ test execution successfully completed.\n20:17:57,145  INFO [main] - \n-------------------------------------------------\nREAD test statistics\nREAD messages: 1974957\nREAD errors: 0, 0.00%\nREAD speed: 6404 msg/sec\n-------------------------------------------------\n20:17:57,207  INFO [main] - Running BULK_READ test\n20:17:57,207  INFO [main] - Setting up load tests driver\n20:17:59,495  INFO [main] - Load tests driver setup successfully completed\n20:17:59,495  INFO [main] - Starting workers\n20:17:59,515  INFO [main] - Workers started\n20:17:59,515  INFO [main] - Waiting for workers to complete\n20:25:59,568  INFO [main] - Worker BULK_READ-worker-0 successfully completed\n20:25:59,568  INFO [main] - Worker BULK_READ-worker-1 successfully completed\n20:25:59,568  INFO [main] - Worker BULK_READ-worker-2 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-3 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-4 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-5 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-6 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-7 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-8 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-9 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-10 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-11 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-12 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-13 successfully completed\n20:25:59,585  INFO [main] - Worker BULK_READ-worker-14 successfully completed\n20:25:59,586  INFO [main] - Worker BULK_READ-worker-15 successfully completed\n20:25:59,586  INFO [main] - Worker BULK_READ-worker-16 successfully completed\n20:25:59,586  INFO [main] - Worker BULK_READ-worker-17 successfully completed\n20:25:59,586  INFO [main] - Worker BULK_READ-worker-18 successfully completed\n20:25:59,586  INFO [main] - Worker BULK_READ-worker-19 successfully completed\n20:25:59,586  INFO [main] - BULK_READ test execution successfully completed.\n20:25:59,586  INFO [main] - \n-------------------------------------------------\nBULK_READ test statistics\nBULK_READ messages: 3832300\nBULK_READ errors: 0, 0.00%\nBULK_READ speed: 12790 msg/sec\n-------------------------------------------------\n20:25:59,653  INFO [main] - Ignite load tests execution completed\n",
      "language": "text"
    }
  ]
}
[/block]
According to the provided log we can see that:
* Average speed of single `WRITE` operation is `5597 msg/sec`
* Average speed of `BULK_WRITE` operation is `6748 msg/sec`
* Average speed of single `READ` operation is `6404 msg/sec`
* Average speed of `BULK_READ` operation is `12790 msg/sec`
* There were no errors occurred for each kind of tests. When you simulating load, which is higher than your current Ignite/Cassandra infrastructure can handle, some of the `WRITE/BULK_WRITE/READ/BULK_READ` operations can fail and it will be reflected in the number (and percentage) of errors in the tests statistics.

Thus to simulate the real load for your cluster you just need to run the same load test simultaneously from multiple client nodes. After that just summarize average speed of each test among all the nodes and it will be the average speed of the `READ/WRITE/BULK_READ/BULK_WRITE` operation which your current configuration could handle.
[block:callout]
{
  "type": "info",
  "body": "If you plan to run load tests using [AWS infrastructure](https://aws.amazon.com/products/?nc2=h_ql_sf_ls), you can just use [AWS infrastructure deployment](doc:aws-infrastructure-deployment), which automatically takes care about all the routine (create and bootstrap required amount of EC2 instances for `Ignite/Cassandra/Tests` clusters, run load tests and wait for their completion, collect all the load tests statistics from each EC2 instance and produce summary report). As a bonus you'll also have `Cassandra/Ignite/Tests` clusters monitoring based on [Ganglia](http://ganglia.info/) which allows you to see what's going on with your clusters under high load."
}
[/block]

[block:callout]
{
  "type": "info",
  "body": "It's recommended to execute **recreate-cassandra-artifacts.sh / recreate-cassandra-artifacts.bat** script, before running load tests. The script will clean up all the Cassandra keyspace/tables which exists from the previous load tests execution. Otherwise statistics could be not very accurate. If you are using [AWS infrastructure deployment](doc:aws-infrastructure-deployment) this will be done for you automatically."
}
[/block]