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

After building Ignite distribution from the source code you will be able to find `target/tests-package` directory inside Cassandra module directory (and `target/ignite-cassandra-tests-<current-version>.zip` archive of this directory). This directory contains ready to use load tests application for Ignite Cassandra module and has such structure:

* **lib** - directory containing all the required **jars** to communicate with Ignite and Cassandra. If you are going to run load tests for your custom key/value classes, you should create a separate **jar** for them and put it inside this directory (see more details in the next chapters).

* **settings** - directory containing load tests configuration settings like Cassandra and Ignite connection details, persistent settings and so on (see more details in the next chapters).

* **cassandra-load-tests.bat** - shell script for MS Windows, to run persistence load tests directly against Cassandra cluster. This test allows you to measure the performance of direct key/value persistence operations for your Cassandra cluster bypassing Ignite cluster. This could be rather useful, cause based on the test results you'll be able to select Cassandra cluster of appropriate capacity to be used as a persistence store for your Ignite cache.

* **cassandra-load-tests.sh** - the same shell script like above, but for Linux. 

* **ignite-load-tests.bat** - shell script for MS Windows, to run persistence load tests against Ignite cluster. This test allows you to measure the performance of key/value persistence operations for your Ignite cluster with a Cassandra cluster configured to be used as a persistence store. Based on the test results you'll be able to select Ignite cluster of appropriate capacity.

* **ignite-load-tests.sh** - the same shell script like above, but for Linux.

As you just seen there are actually two types of load tests:
* **Cassandra load tests** - performs all key/value persistence operations directly with Cassandra bypassing Ignite.
* **Ignite load tests** - performs all key/value persistence operations with Ignite using appropriate instance of [IgniteCache](https://apacheignite.readme.io/docs/getting-started#first-ignite-data-grid-application).

Now lets take a look at load tests scenarios and configuration details.