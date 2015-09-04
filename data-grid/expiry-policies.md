Expiry Policy specifies the amount of time that must pass before a cache entry is considered expired. Time can be counted from creation, last access or modification time.

Expiry Policy can be setup using one of predefined implementations of `ExpiryPolicy`.
Comparision of predefined implementations:
[block:parameters]
{
  "data": {
    "0-0": "`CreatedExpiryPolicy`",
    "0-1": "Yes",
    "1-0": "`AccessedExpiryPolicy`",
    "1-1": "Yes",
    "2-0": "`ModifiedExpiryPolicy`",
    "2-1": "Yes",
    "3-0": "`TouchedExpiryPolicy`",
    "3-1": "Yes",
    "h-0": "Class name",
    "h-1": "creation time",
    "4-0": "`EternalExpiryPolicy`",
    "4-1": "No",
    "h-2": "last access time",
    "h-3": "last update time",
    "0-2": "No",
    "0-3": "No",
    "1-2": "Yes",
    "1-3": "No",
    "2-3": "Yes",
    "2-2": "No",
    "3-2": "Yes",
    "3-3": "Yes",
    "4-2": "No",
    "4-3": "No"
  },
  "cols": 4,
  "rows": 5
}
[/block]
Custom `ExpiryPolicy` implementations also allowed.

Expiry Policy can be setup at `CacheConfiguration`. This policy will be used for all entries inside cache.
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
Also, it is possible to change or set Expiry Policy for some operations on cache. 
[block:code]
{
  "codes": [
    {
      "code": "IgniteCache<Object, Object> c = cache.withExpiryPolicy(\n\t\tnew CreatedExpiryPolicy(new Duration(TimeUnit.SECONDS, 5)));",
      "language": "java"
    }
  ]
}
[/block]
This policy will be used for each operation invoked on the returned cache.