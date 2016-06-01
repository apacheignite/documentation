One of the benefits of Ignite Cassandra module is that - you don't need to care about Cassandra DDL syntax for table creation and Java to Cassandra type mapping details. 

You just need to build XML configuration document ([persistence descriptor](doc:base-concepts#persistencesettingsbean) which specifies how Ignite cache keys and values should be serialized/deserialized to/from Cassandra. Based on this settings all the absent Cassandra keyspaces and tables will be created automatically. The only requirement for all this "magic" to work:
[block:callout]
{
  "type": "danger",
  "title": "IN THE CONNECTION SETTINGS FOR CASSANDRA, YOU SHOULD SPECIFY USER HAVING ENOUGH PERMISSIONS TO CREATE KEYSPACES/TABLES"
}
[/block]