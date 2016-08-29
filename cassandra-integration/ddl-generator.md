One of the benefits of Ignite Cassandra module is that - you don't need to care about Cassandra DDL syntax for table creation and Java to Cassandra type mapping details. 

You just need to build XML configuration document ([persistence descriptor](doc:base-concepts#persistencesettingsbean)) which specifies how Ignite cache keys and values should be serialized/deserialized to/from Cassandra. Based on this settings all the absent Cassandra keyspaces and tables will be created automatically. The only requirement for all this "magic" to work:
[block:callout]
{
  "type": "danger",
  "body": "In connection settings for Cassandra, you should specify user having enough permissions to create keyspaces/tables"
}
[/block]
However, for some of the deployments it's not possible - because of the very strict security policy. Thus the only solution in a such situation is to provide DDL scripts for DevOps team to create all the necessary Cassandra keyspaces/tables in advance. 

That's the exact use-case for DDL generator utility - it generates DDLs from [persistence descriptors](doc:base-concepts#persistencesettingsbean)

Syntax sample:
[block:code]
{
  "codes": [
    {
      "code": "java org.apache.ignite.cache.store.cassandra.utils.DDLGenerator /opt/dev/ignite/persistence-settings-1.xml /opt/dev/ignite/persistence-settings-2.xml",
      "language": "shell"
    }
  ]
}
[/block]
Output sample:
[block:code]
{
  "codes": [
    {
      "code": "-------------------------------------------------------------\nDDL for keyspace/table from file: /opt/dev/ignite/persistence-settings-1.xml\n-------------------------------------------------------------\n\ncreate keyspace if not exists test1\nwith replication = {'class' : 'SimpleStrategy', 'replication_factor' : 3} and durable_writes = true;\n\ncreate table if not exists test1.primitive_test1\n(\n key int,\n value int,\n primary key ((key))\n);\n\n-------------------------------------------------------------\nDDL for keyspace/table from file: /opt/dev/ignite/persistence-settings-2.xml\n-------------------------------------------------------------\n\ncreate keyspace if not exists test1\nwith REPLICATION = {'class' : 'SimpleStrategy', 'replication_factor' : 3} AND DURABLE_WRITES = true;\n\ncreate table if not exists test1.pojo_test3\n(\n company text,\n department text,\n number int,\n first_name text,\n last_name text,\n age int,\n married boolean,\n height bigint,\n weight float,\n birth_date timestamp,\n phones blob,\n primary key ((company, department), number)\n) \nwith comment = 'A most excellent and useful table' AND read_repair_chance = 0.2 and clustering order by (number desc);",
      "language": "text"
    }
  ]
}
[/block]
Just don't forget to set **CLASSPATH** environment variable correctly:

1. Include jar file for Ignite Cassandra module (`ignite-cassandra-<version-number>.jar`) in your CLASSPATH.
2. If you are using **POJO** persistence strategy for some of your custom java classes you need to include jars with these classes in your CLASSPATH as well.