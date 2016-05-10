Given that the most common ways to cache data is in `PARTITIONED` caches, collocating compute with data or data with data can significantly improve performance and scalability of your application.
[block:api-header]
{
  "type": "basic",
  "title": "Collocate Data with Data"
}
[/block]
In many cases it is beneficial to collocate different cache keys together if they will be accessed together. Quite often your business logic will require access to more than one cache key. By collocating them together you can make sure that all keys with the same `affinityKey` will be cached on the same processing node, hence avoiding costly network trips to fetch data from remote nodes.

For example, let's say you have `Person` and `Company` objects and you want to collocate `Person` objects with `Company` objects for which this person works. There are two ways to declare affinity key for a key type.

First, cache key used to cache `Person` objects may have a field or method annotated with `@AffinityKeyMapped` annotation, which will provide the value of the company key for collocation. For convenience, you can also optionally use `AffinityKey` class.
[block:callout]
{
  "type": "info",
  "title": "Annotations in Scala",
  "body": "Note that if Scala case class is used as a key class and one of its constructor parameters is annotated with `@AffinityKeyMapped`, by default the annotation will not be properly applied to the generated field, and therefore will not be recognized by Ignite. To override this behavior, use `@field` [meta annotation](http://www.scala-lang.org/api/current/#scala.annotation.meta.package) in addition to `@AffinityKeyMapped` (see example below)."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "public class PersonKey {\n    // Person ID used to identify a person.\n    private String personId;\n \n    // Company ID which will be used for affinity.\n    @AffinityKeyMapped\n    private String companyId;\n    ...\n}\n\n// Instantiate person keys with the same company ID which is used as affinity key.\nObject personKey1 = new PersonKey(\"myPersonId1\", \"myCompanyId\");\nObject personKey2 = new PersonKey(\"myPersonId2\", \"myCompanyId\");\n \nPerson p1 = new Person(personKey1, ...);\nPerson p2 = new Person(personKey2, ...);\n \n// Both, the company and the person objects will be cached on the same node.\ncompCache.put(\"myCompanyId\", new Company(...));\nperCache.put(personKey1, p1);\nperCache.put(personKey2, p2);",
      "language": "java",
      "name": "using PersonKey"
    },
    {
      "code": "case class PersonKey (\n    // Person ID used to identify a person.\n    personId: String,\n \n    // Company ID which will be used for affinity.\n    @(AffinityKeyMapped @field)\n    companyId: String\n)\n\n// Instantiate person keys with the same company ID which is used as affinity key.\nval personKey1 = PersonKey(\"myPersonId1\", \"myCompanyId\");\nval personKey2 = PersonKey(\"myPersonId2\", \"myCompanyId\");\n \nval p1 = new Person(personKey1, ...);\nval p2 = new Person(personKey2, ...);\n \n// Both, the company and the person objects will be cached on the same node.\ncompCache.put(\"myCompanyId\", Company(...));\nperCache.put(personKey1, p1);\nperCache.put(personKey2, p2);",
      "language": "scala",
      "name": "using PersonKey (Scala)"
    },
    {
      "code": "Object personKey1 = new AffinityKey(\"myPersonId1\", \"myCompanyId\");\nObject personKey2 = new AffinityKey(\"myPersonId2\", \"myCompanyId\");\n \nPerson p1 = new Person(personKey1, ...);\nPerson p2 = new Person(personKey2, ...);\n \n// Both, the company and the person objects will be cached on the same node.\ncomCache.put(\"myCompanyId\", new Company(..));\nperCache.put(personKey1, p1);\nperCache.put(personKey2, p2);",
      "language": "java",
      "name": "using AffinityKey"
    }
  ]
}
[/block]
Second, cache key configuration may be declared in IgniteConfiguration object specifying which field should be used as an affinity key field.
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <property name=\"cacheKeyConfiguration\">\n        <bean class=\"org.apache.ignite.cache.CacheKeyConfiguration\">\n            <property name=\"typeName\" value=\"org.apache.ignite.examples.PersonKey\"/>\n            <property name=\"affinityKeyFieldName\" value=\"companyId\"/>\n        </bean>\n    </property>\n...\n</bean>",
      "language": "xml",
      "name": "Using CacheKeyConfiguration"
    }
  ]
}
[/block]

[block:callout]
{
  "type": "info",
  "title": "SQL Joins",
  "body": "When performing [SQL distributed joins](/docs/cache-queries#sql-queries) over data residing in partitioned caches, you must make sure that the join-keys are collocated."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Collocating Compute with Data"
}
[/block]
It is also possible to route computations to the nodes where the data is cached. This concept is known as Collocation Of Computations And Data. It allows to route whole units of work to a certain node. 

To collocate compute with data you should use `IgniteCompute.affinityRun(...)` and `IgniteCompute.affinityCall(...)` methods.

Here is how you can collocate your computation with the same cluster node on which company and persons from the example above are cached.
[block:code]
{
  "codes": [
    {
      "code": "String companyId = \"myCompanyId\";\n \n// Execute Runnable on the node where the key is cached.\nignite.compute().affinityRun(\"myCache\", companyId, () -> {\n  Company company = compCache.get(companyId);\n\n  // Since we collocated persons with the company in the above example,\n  // access to the persons objects is local.\n  Person person1 = perCache.get(personKey1);\n  Person person2 = perCache.get(personKey2);\n  ...  \n});",
      "language": "java",
      "name": "affinityRun"
    },
    {
      "code": "final String companyId = \"myCompanyId\";\n \n// Execute Runnable on the node where the key is cached.\nignite.compute().affinityRun(\"myCache\", companyId, new IgniteRunnable() {\n  @Override public void run() {\n    Company company = compCache.get(companyId);\n    \n    Person person1 = perCache.get(personKey1);\n    Person person2 = perCache.get(personKey2);\n    ...\n  }\n};",
      "language": "java",
      "name": "java7 affinityRun"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "IgniteCompute vs EntryProcessor"
}
[/block]
Both, `IgniteCompute.affinityRun(...)` and `IgniteCache.invoke(...)` methods offer ability to collocate compute and data. The main difference is that `invoke(...)` methods is atomic and executes while holding a lock on a key. You should not access other keys from within the `EntryProcessor` logic as it may cause a deadlock. 

 `affinityRun(...)` and `affinityCall(...)`, on the other hand, do not hold any locks. For example, it is absolutely legal to start multiple transactions or execute cache queries from these methods without worrying about deadlocks. In this case Ignite will automatically detect that the processing is collocated and will employ a light-weight 1-Phase-Commit optimization for transactions (instead of 2-Phase-Commit).
[block:callout]
{
  "type": "info",
  "body": "See [JCache EntryProcessor](/docs/jcache#entryprocessor) documentation for more information about `IgniteCache.invoke(...)` method."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Affinity Function"
}
[/block]
Affinity of a partition controls which grid node or nodes a partition will be cached on. `AffinityFunction` is a pluggable API used to determine an ideal mapping of partitions to nodes in the grid. When cluster topology changes, the partition-to node mapping may be different from an ideal distribution provided by the affinity function until rebalancing is completed.
Ignite is shipped with two predefined affinity function implementations:
 * `RendezvousAffinityFunction`. This function allows a bit of discrepancy in partition-to-node mapping (i.e. some nodes may be responsible for a slightly larger number of partitions than others), however, it guarantees that when topology changes, partitions are migrated only to a joined node or only from a left node. No data exchange will happen between existing nodes in a cluster.
 * `FairAffinityFunction`. This functions tries to make sure that partition distribution among cluster nodes is even. This comes at a price of a possible partition migration between existing nodes in a cluster.

Note that the cache affinity function does not directly map keys to nodes, it maps keys to partitions. Partition is simply a number from a limited set (0 to 1024 by default). After the keys are mapped to their partitions (i.e. they get their partition numbers), the existing partition-to-nodes mapping is used for current topology version. Key-to-partition mapping must not change over the time.
[block:callout]
{
  "type": "info",
  "title": "Crash-safe Affinity",
  "body": "It is useful to arrange partitions in a cluster in such a way that primary and backup copies are not located on the same physical machine. To ensure this property, a user can set `excludeNeighbors` flag on both `RendezvousAffinityFunction` and `FairAffinityFunction`.\n\nSometimes it is also useful to have primary and backup copies of a partition on a different racks. In this case a user may assign a specific attribute to each node and then use `backupFilter` property on both `RendezvousAffinityFunction` and `FairAffinityFunction` to exclude nodes from the same rack from candidates for backup copy assignment."
}
[/block]
`AffinityFunction` is a pluggable API and a user can provide it's own implementation of the function. The 3 main methods of `AffinityFunction` API are:
 * partitions() - Given the total number of partitions for a cache. Cannot be changed while cluster is up.
 * partition(...) - Given a key, this method determines which partition a key belongs to. The mapping must not change over time.
assignPartitions(...) - This method is called every time a cluster topology changes. This method returns a partition-to-node mapping for the given cluster topology.
[block:api-header]
{
  "type": "basic",
  "title": "Affinity Key Mapper"
}
[/block]

`CacheAffinityKeyMapper` is a pluggable API responsible for getting an affinity key for a cache key. Usually cache key itself is used for affinity, however sometimes it is important to change affinity of a cache key in order to collocate it with other cache keys.

The main method of `CacheAffinityKeyMapper` is `affinityKey(key)` which returns `affinityKey` for a cache key. By default, Ignite will look for any field or method annotated with `@CacheAffinityKeyMapped` annotation. If such field or method is not found, then the cache key itself is used for affinity. If such field or method is found, then the value of this field or method will be returned from `CacheAffinityKeyMapper.affinityKey(key)`method. This allows you to specify an alternate affinity key, other than the cache key itself, whenever needed.