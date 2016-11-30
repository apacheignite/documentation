* [Overview](#overview)
* [Affinity Call and Run Methods](#affinity-call-and-run-methods) 
[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
Collocation of computations with data allow for minimizing data serialization within network and can significantly improve performance and scalability of your application. Whenever possible, you should always make best effort to colocate your computations with the cluster nodes caching the data that needs to be processed.
[block:api-header]
{
  "type": "basic",
  "title": "Affinity Call and Run Methods"
}
[/block]
`affinityCall(...)`  and `affinityRun(...)` methods co-locate jobs with nodes on which data is cached. In other words, knowing a cache name and affinity key these methods will be able to find the node that is the primary for the given key and will execute a job there. 
[block:callout]
{
  "type": "info",
  "title": "Consistency Guarantee",
  "body": "Starting from Apache Ignite 1.8 it's guaranteed that the partition, which the affinity key belongs to, will not be evicted from a node while a job, triggered by `affinityCall(...)` or `affinityRun(...)`, will be being executed there. The partition rebalancing usually happens due to the topology change event when a new node joins the cluster or the old one leaves it.\n\nThis guarantee makes it feasible to execute complex logic for which it's crucial that the data stays on the same node throughout the time the job is being executed there. For instance, this feature allows executing *local* SQL queries as a part of a compute job, triggered by `affinityCall(...)` or `affinityRun(...)`, not worrying about that the local query may return a partial result set due to the data rebalancing."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Integer, String> cache = ignite.cache(CACHE_NAME);\n\nIgniteCompute compute = ignite.compute();\n\nfor (int key = 0; key < KEY_CNT; key++) {\n    // This closure will execute on the remote node where\n    // data with the 'key' is located.\n    compute.affinityRun(CACHE_NAME, key, () -> { \n        // Peek is a local memory lookup.\n        System.out.println(\"Co-located [key= \" + key + \", value= \" + cache.localPeek(key) +']');\n    });\n}",
      "language": "java",
      "name": "affinityRun"
    },
    {
      "code": "IgniteCache<Integer, String> cache = ignite.cache(CACHE_NAME);\n\nIgniteCompute asyncCompute = ignite.compute().withAsync();\n\nList<IgniteFuture<?>> futs = new ArrayList<>();\n\nfor (int key = 0; key < KEY_CNT; key++) {\n    // This closure will execute on the remote node where\n    // data with the 'key' is located.\n    asyncCompute.affinityRun(CACHE_NAME, key, () -> { \n        // Peek is a local memory lookup.\n        System.out.println(\"Co-located [key= \" + key + \", value= \" + cache.peek(key) +']');\n    });\n  \n    futs.add(asyncCompute.future());\n}\n\n// Wait for all futures to complete.\nfuts.stream().forEach(IgniteFuture::get);",
      "language": "java",
      "name": "async affinityRun"
    },
    {
      "code": "final IgniteCache<Integer, String> cache = ignite.cache(CACHE_NAME);\n\nIgniteCompute compute = ignite.compute();\n\nfor (int i = 0; i < KEY_CNT; i++) {\n    final int key = i;\n \n    // This closure will execute on the remote node where\n    // data with the 'key' is located.\n    compute.affinityRun(CACHE_NAME, key, new IgniteRunnable() {\n        @Override public void run() {\n            // Peek is a local memory lookup.\n            System.out.println(\"Co-located [key= \" + key + \", value= \" + cache.peek(key) +']');\n        }\n    });\n}",
      "language": "java",
      "name": "java7 affinityRun"
    }
  ]
}
[/block]