Ignite Cassandra module provides a set of unit tests, which allows you to check that such functionality works well:

1. `PRIMITIVE` persistence strategy tests by directly serializing/deserializing primitive java types (int, long, float and etc.) into Cassandra bypassing Ignite.
2. `BLOB` persistence strategy tests by directly serializing/deserializing primitive and custom java types into Cassandra bypassing Ignite and using Java and Kryo serialization system.
3. `POJO` persistence strategy tests by directly serializing/deserializing custom java types into Cassandra bypassing Ignite.
4. `PRIMITIVE` persistence strategy tests through Ignite cache.
5. `BLOB` persistence strategy tests through Ignite cache.
5. `POJO` persistence strategy tests through Ignite cache.
6. Ignite cache warm up through `loadCache` operation.