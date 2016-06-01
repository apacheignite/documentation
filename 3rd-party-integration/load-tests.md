Ignite Cassandra module provides a set of load tests which allows you to simulate production load for Ignite and Cassandra with your custom key/value classes and persistence settings. Thus using these load tests you can measure performance characteristics of the system for your specific configuration:
* Set of custom key/value classes
* Set of Ignite nodes
* Set of Cassandra nodes

Such kind of measurements could be useful for better understanding of system scalability and sizing.
[block:api-header]
{
  "type": "basic",
  "title": "Building load tests"
}
[/block]
Load tests for Cassandra module are provided as a part of tests source code of the module. Thus first of all you should [build Ignite distribution](doc:getting-started#section-building-from-source) from the source code. 

After building Ignite distribution from the source code you will be able to find `target/tests-package` directory inside Cassandra module directory (and `target/ignite-cassandra-tests-<version>.zip` archive of this directory as well). The directory contains ready to use load tests application for Ignite Cassandra module and has such structure:

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
  "title": "Load tests settings"
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
  "title": "Running load tests"
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
  "title": "Using custom key/value classes"
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
  "title": "Analyzing tests execution results"
}
[/block]
The results of load tests execution are provided as `log4j` log files. By default (if you didn't modify `log4j.properties` file) there are two files which shows summary results of tests execution:

1. **cassandra-load-tests.log** - contains summary statistic of Cassandra load tests execution results (if you run `cassandra-load-tests.sh` or `cassandra-load-tests.bat` shell script).
2. **ignite-load-tests.log** - contains summary statistic of Ignite load tests execution results (if you run `ignite-load-tests.sh` or `ignite-load-tests.bat` shell script).

Here is an example of `cassandra-load-tests.log`: