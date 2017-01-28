Expiry Policy specifies the amount of time that must pass before a cache entry is considered expired. Time can be counted from creation, last access or modification time.

Expiry Policy can be setup using any of the predefined implementations of `ExpiryPolicy`:
[block:parameters]
{
  "data": {
    "0-0": "`CreatedExpiryPolicy`",
    "0-1": "Used",
    "1-0": "`AccessedExpiryPolicy`",
    "1-1": "Used",
    "2-0": "`ModifiedExpiryPolicy`",
    "2-1": "Used",
    "3-0": "`TouchedExpiryPolicy`",
    "3-1": "Used",
    "h-0": "Class name",
    "h-1": "creation time",
    "4-0": "`EternalExpiryPolicy`",
    "4-1": "",
    "h-2": "last access time",
    "h-3": "last update time",
    "0-2": "",
    "0-3": "",
    "1-2": "Used",
    "1-3": "",
    "2-3": "Used",
    "2-2": "",
    "3-2": "Used",
    "3-3": "Used",
    "4-2": "",
    "4-3": ""
  },
  "cols": 4,
  "rows": 5
}
[/block]
Custom `ExpiryPolicy` implementations are also allowed.

Expiry Policy can be setup in `CacheConfiguration`. This policy will be used for all entries inside cache.
[block:code]
{
  "codes": [
    {
      "code": "cfg.setExpiryPolicyFactory(CreatedExpiryPolicy.factoryOf(Duration.ZERO));",
      "language": "java"
    }
  ]
}
[/block]
Also, it is possible to change or set Expiry Policy for individual operations on cache. 
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Object, Object> cache = cache.withExpiryPolicy(\n\t\tnew CreatedExpiryPolicy(new Duration(TimeUnit.SECONDS, 5)));",
      "language": "java"
    }
  ]
}
[/block]
This policy will be used for each operation invoked on the returned cache instance.
[block:api-header]
{
  "type": "basic",
  "title": "Eager TTL"
}
[/block]
Entries that are expired can be removed from cache either eagerly or when they are touched by different cache operations. If there is at least one cache configured with eager TTL enabled, Ignite will create a single thread to clean up expired entries in background.

Eager TTL can be enabled or disabled via `CacheConfiguration.eagerTtl` property (default value is `true`):
[block:code]
{
  "codes": [
    {
      "code": "<bean class=\"org.apache.ignite.configuration.CacheConfiguration\">\n    <property name=\"eagerTtl\" value=\"true\"/>\n</bean>",
      "language": "xml"
    }
  ]
}
[/block]