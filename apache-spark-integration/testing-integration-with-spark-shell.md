[block:api-header]
{
  "type": "basic",
  "title": "Starting up the cluster"
}
[/block]
Here we will briefly cover the process of Spark and Ignite cluster startup. Refer to [Spark documentation](https://spark.apache.org/docs/latest/) for more details.

For the testing you will need a Spark master process and at least one Spark worker. Usually Spark master and workers are separate machines, but for the test purposes you can start worker on the same machine where master starts.

1. Download and unpack Spark binary distribution to the same location (let it be `SPARK_HOME`) on all nodes.
3. Download and unpack Ignite binary distribution to the same location (let it be `IGNITE_HOME`) on all nodes.
3. On master node `cd` to `$SPARK_HOME` and run the following command:
[block:code]
{
  "codes": [
    {
      "code": "sbin/start-master.sh",
      "language": "shell"
    }
  ]
}
[/block]
The script should output the path to log file of the started process. Check the log file for the master URL which has the following format: `spark://master_host:master_port` Also check the log file for the Web UI url (usually it is `http://master_host:8080`).

4. On each of the worker nodes `cd` to `$SPARK_HOME` and run the following command:
[block:code]
{
  "codes": [
    {
      "code": "bin/spark-class org.apache.spark.deploy.worker.Worker spark://master_host:master_port",
      "language": "shell"
    }
  ]
}
[/block]
where `spark://master_host:master_port` is the master URL you grabbed from the master log file. After workers has started check the master Web UI interface, it should show all of your workers registered in status `ALIVE`

5. On each of the worker nodes `cd` to `$IGNITE_HOME` and start an Ignite node by running the following command:
[block:code]
{
  "codes": [
    {
      "code": "bin/ignite.sh",
      "language": "shell"
    }
  ]
}
[/block]
You should see Ignite nodes discover each other with default configuration. If your network does not allow multicast traffic, you will need to change the default configuration file and configure TCP discovery.
[block:api-header]
{
  "type": "basic",
  "title": "Working with spark-shell"
}
[/block]
Now, after you have your cluster up and running, you can run `spark-shell` and check the integration.

1. Start spark shell:
 
- Either by providing Maven coordinates to Ignite artifacts (make sure to use `--repositories` if working with staging or snapshot repository, otherwise `--repositories` may be omitted):
[block:code]
{
  "codes": [
    {
      "code": "./bin/spark-shell \n\t--packages org.apache.ignite:ignite-spark:1.1.2-SNAPSHOT\n  --repositories https://repository.apache.org/content/repositories/snapshots\n  --master spark://master_host:master_port",
      "language": "shell"
    }
  ]
}
[/block]
- Or by providing paths to Ignite jar file paths using `--jars` parameter
[block:code]
{
  "codes": [
    {
      "code": "./bin/spark-shell --jars path/to/ignite-core.jar,path/to/ignite-spark.jar,path/to/cache-api.jar,path/to/ignite-log4j.jar,path/to/log4j.jar --master spark://master_host:master_port",
      "language": "shell"
    }
  ]
}
[/block]
You should see Spark shell started up. 

2. Let's create an instance of Ignite context using default configuration:
[block:code]
{
  "codes": [
    {
      "code": "import org.apache.ignite.spark._\nimport org.apache.ignite.configuration._\n\nval ic = new IgniteContext[Integer, Integer](sc, () => new IgniteConfiguration())",
      "language": "scala"
    }
  ]
}
[/block]
You should see something like 
[block:code]
{
  "codes": [
    {
      "code": "ic: org.apache.ignite.spark.IgniteContext[Integer,Integer] = org.apache.ignite.spark.IgniteContext@62be2836",
      "language": "text",
      "name": null
    }
  ]
}
[/block]
3. Let's now create an instance of `IgniteRDD` using "partitioned" cache in default configuration:
[block:code]
{
  "codes": [
    {
      "code": "val cache = ic.fromCache(\"partitioned\")",
      "language": "scala"
    }
  ]
}
[/block]
You should see an instance of RDD created for partitioned cache:
[block:code]
{
  "codes": [
    {
      "code": "cache: org.apache.ignite.spark.IgniteRDD[Integer,Integer] = IgniteRDD[0] at RDD at IgniteAbstractRDD.scala:27",
      "language": "text"
    }
  ]
}
[/block]
Note that creation of RDD is a local operation and will not create a cache in Ignite cluster. 

4. Let's now actually ask Spark to do something with our RDD, for example, get all pairs where value is less than 10:
[block:code]
{
  "codes": [
    {
      "code": "cache.filter(_._2 < 10).collect()",
      "language": "scala"
    }
  ]
}
[/block]
As our cache has not been filled yet, the result will be an empty array:
[block:code]
{
  "codes": [
    {
      "code": "res0: Array[(Integer, Integer)] = Array()",
      "language": "text"
    }
  ]
}
[/block]
Check the logs of remote spark workers and see how Ignite context will start clients on all remote workers in the cluster. You can also start command-line Visor and check that "partitioned" cache has been created.

5. Let's now save some values into Ignite:
[block:code]
{
  "codes": [
    {
      "code": "cache.savePairs(sc.parallelize(1 to 100000, 10).map(i => (i, i)))",
      "language": "scala"
    }
  ]
}
[/block]
After running this command you can check with command-line Visor that cache size is 100000 elements. 

6. We can now check how the state we created will survive job restart. Shut down the spark shell and repeat steps 1-3. You should again have an instance of Ignite context and RDD for "partitioned" cache. We can now check how many keys there are in our RDD which value is greater than 50000:
[block:code]
{
  "codes": [
    {
      "code": "cache.filter(_._2 > 50000).count",
      "language": "scala"
    }
  ]
}
[/block]
Since we filled up cache with a sequence of number from 1 to 100000 inclusive, we should see `50000` as a result:
[block:code]
{
  "codes": [
    {
      "code": "res0: Long = 50000",
      "language": "text"
    }
  ]
}
[/block]