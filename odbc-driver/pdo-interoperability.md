[block:api-header]
{
  "type": "basic",
  "title": "Overview"
}
[/block]
PHP provides a lightweight, consistent interface for accessing databases named PHP Data Objects   - PDO. This extension goes with several database-specific PDO drivers. One of them is [PDO_ODBC](http://php.net/manual/en/ref.pdo-odbc.php), which allows connecting to any database that provides its own ODBC driver.

With the usage of Apache Ignite's ODBC driver it's possible to connect to an Apache Ignite cluster from a PHP application accessing and modifying data that is stored there. This page provides instructions on how to bring this to life.
[block:api-header]
{
  "type": "basic",
  "title": "Setting Up ODBC Driver"
}
[/block]
Apache Ignite conforms to ODBC protocol and has its own ODBC driver that is released along with other functionality. This is the driver that will be used by PHP PDO framework going forward in order to connect to an Apache Ignite cluster.

Refer to Apache Ignite [ODBC documentation](doc:odbc-driver) configuring it and installing on a target system. Once the driver is installed and works fine move to the section below of this guide.
[block:callout]
{
  "type": "warning",
  "title": "",
  "body": "Use ODBC driver that is available under Apache Ignite 1.8 and later versions. The prior versions of the driver and Apache Ignite don't support PHP PDO framework."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Installing and Configuring PHP PDO"
}
[/block]
To install PHP, PDO and PDO_ODBC driver refer to the generic PHP resources:
* [Download](http://php.net/downloads.php) and install the desired PHP version. Note, that PDO driver is enabled by default in PHP as of PHP 5.1.0.
* [Configure](http://php.net/manual/en/book.pdo.php) PHP PDO framework.
* [Enable](http://php.net/manual/en/ref.pdo-odbc.php) PDO_ODBC driver.
* If necessary, [configure](http://php.net/manual/en/ref.pdo-odbc.php#ref.pdo-odbc.installation) PDO_ODBC driver for your system. In most cases, however, simple installation of the PDO_ODBC driver is going to be enough.
[block:api-header]
{
  "type": "basic",
  "title": "Starting Ignite Cluster"
}
[/block]
After PHP PDO is installed and ready to be used let's start an Ignite cluster with an exemplary configuration and connect to the cluster from a PHP application updating and querying cluster's data.

First of all, you should enable ODBC processor on the node, you are going to connect to. To do so you need to specify

[block:code]
{
  "codes": [
    {
      "code": "CONFIGURATION THAT IS USED in the example",
      "language": "xml",
      "name": "XML"
    },
    {
      "code": "/**\n * Person class.\n */\npublic class Person implements Serializable {\n  @QuerySqlField\n  public String firstName;\n\n  @QuerySqlField\n  public String lastName;\n\n  @QuerySqlField\n  public String resume;\n\n  @QuerySqlField(index = true)\n  public double salary;\n\n  public Person() {\n    // No-op.\n  }\n\n  /**\n   * Constructs person record.\n   *\n   * @param org       Organization.\n   * @param firstName First name.\n   * @param lastName  Last name.\n   * @param salary    Salary.\n   * @param resume    Resume text.\n   */\n  public Person(String firstName, String lastName, double salary, String resume) {\n    this.firstName = firstName;\n    this.lastName = lastName;\n    this.salary = salary;\n    this.resume = resume;\n  }\n}",
      "language": "java",
      "name": "Person class"
    },
    {
      "code": "public class Example {\n\n  public static void main(String[] args) throws Exception {\n    try (Ignite ignite = Ignition.start(\"example.xml\")) {\n      \n      CacheConfiguration<Long, Person> personCacheCfg = new CacheConfiguration<>();\n\n      personCacheCfg.setCacheMode(CacheMode.PARTITIONED); // Default.\n      personCacheCfg.setIndexedTypes(Long.class, Person.class);\n\n      // Auto-close cache at the end of the example.\n      try (\n        IgniteCache<Long, Organization> orgCache = ignite.getOrCreateCache(\"Person\");\n        IgniteCache<AffinityKey<Long>, Person> colPersonCache = ignite.getOrCreateCache(colPersonCacheCfg);\n        IgniteCache<AffinityKey<Long>, Person> personCache = ignite.getOrCreateCache(personCacheCfg)\n      ) {\n        // Populate cache.\n        initialize();\n\n        // Example for SCAN-based query based on a predicate.\n        scanQuery();\n\n        // Example for SQL-based querying employees based on salary ranges.\n        sqlQuery();\n\n        // Example for SQL-based querying employees for a given organization (includes SQL join for collocated objects).\n        sqlQueryWithJoin();\n\n        // Example for SQL-based querying employees for a given organization (includes distributed SQL join).\n        sqlQueryWithDistributedJoin();\n\n        // Example for TEXT-based querying for a given string in peoples resumes.\n        textQuery();\n\n        // Example for SQL-based querying to calculate average salary among all employees within a company.\n        sqlQueryWithAggregation();\n\n        // Example for SQL-based fields queries that return only required\n        // fields instead of whole key-value pairs.\n        sqlFieldsQuery();\n\n        // Example for SQL-based fields queries that uses joins.\n        sqlFieldsQueryWithJoin();\n      }\n      finally {\n        // Distributed cache could be removed from cluster only by #destroyCache() call.\n        ignite.destroyCache(COLLOCATED_PERSON_CACHE);\n        ignite.destroyCache(PERSON_CACHE);\n        ignite.destroyCache(ORG_CACHE);\n      }\n\n      print(\"Cache query example finished.\");\n    }\n  }\n\n  /**\n     * Example for scan query based on a predicate.\n     */\n  private static void scanQuery() {\n    IgniteCache<BinaryObject, BinaryObject> cache = Ignition.ignite().cache(COLLOCATED_PERSON_CACHE).withKeepBinary();\n\n    ScanQuery<BinaryObject, BinaryObject> scan = new ScanQuery<>(\n      new IgniteBiPredicate<BinaryObject, BinaryObject>() {\n        @Override public boolean apply(BinaryObject key, BinaryObject person) {\n          return person.<Double>field(\"salary\") <= 1000;\n        }\n      }\n    );\n\n    // Execute queries for salary ranges.\n    print(\"People with salaries between 0 and 1000 (queried with SCAN query): \", cache.query(scan).getAll());\n  }\n\n  /**\n     * Example for SQL queries based on salary ranges.\n     */\n  private static void sqlQuery() {\n    IgniteCache<AffinityKey<Long>, Person> cache = Ignition.ignite().cache(PERSON_CACHE);\n\n    // SQL clause which selects salaries based on range.\n    String sql = \"salary > ? and salary <= ?\";\n\n    // Execute queries for salary ranges.\n    print(\"People with salaries between 0 and 1000 (queried with SQL query): \",\n          cache.query(new SqlQuery<AffinityKey<Long>, Person>(Person.class, sql).\n                      setArgs(0, 1000)).getAll());\n\n    print(\"People with salaries between 1000 and 2000 (queried with SQL query): \",\n          cache.query(new SqlQuery<AffinityKey<Long>, Person>(Person.class, sql).\n                      setArgs(1000, 2000)).getAll());\n  }\n\n  /**\n     * Example for SQL queries based on all employees working for a specific organization.\n     */\n  private static void sqlQueryWithJoin() {\n    IgniteCache<AffinityKey<Long>, Person> cache = Ignition.ignite().cache(COLLOCATED_PERSON_CACHE);\n\n    // SQL clause query which joins on 2 types to select people for a specific organization.\n    String joinSql =\n      \"from Person, \\\"\" + ORG_CACHE + \"\\\".Organization as org \" +\n      \"where Person.orgId = org.id \" +\n      \"and lower(org.name) = lower(?)\";\n\n    // Execute queries for find employees for different organizations.\n    print(\"Following people are 'ApacheIgnite' employees: \",\n          cache.query(new SqlQuery<AffinityKey<Long>, Person>(Person.class, joinSql).\n                      setArgs(\"ApacheIgnite\")).getAll());\n\n    print(\"Following people are 'Other' employees: \",\n          cache.query(new SqlQuery<AffinityKey<Long>, Person>(Person.class, joinSql).\n                      setArgs(\"Other\")).getAll());\n  }\n\n  /**\n     * Example for SQL queries based on all employees working for a specific organization (query uses distributed join).\n     */\n  private static void sqlQueryWithDistributedJoin() {\n    IgniteCache<AffinityKey<Long>, Person> cache = Ignition.ignite().cache(PERSON_CACHE);\n\n    // SQL clause query which joins on 2 types to select people for a specific organization.\n    String joinSql =\n      \"from Person, \\\"\" + ORG_CACHE + \"\\\".Organization as org \" +\n      \"where Person.orgId = org.id \" +\n      \"and lower(org.name) = lower(?)\";\n\n    SqlQuery qry = new SqlQuery<AffinityKey<Long>, Person>(Person.class, joinSql).\n      setArgs(\"ApacheIgnite\");\n\n    // Enable distributed joins for query.\n    qry.setDistributedJoins(true);\n\n    // Execute queries for find employees for different organizations.\n    print(\"Following people are 'ApacheIgnite' employees (distributed join): \", cache.query(qry).getAll());\n\n    qry.setArgs(\"Other\");\n\n    print(\"Following people are 'Other' employees (distributed join): \", cache.query(qry).getAll());\n  }\n\n  /**\n     * Example for TEXT queries using LUCENE-based indexing of people's resumes.\n     */\n  private static void textQuery() {\n    IgniteCache<AffinityKey<Long>, Person> cache = Ignition.ignite().cache(PERSON_CACHE);\n\n    //  Query for all people with \"Master Degree\" in their resumes.\n    QueryCursor<Cache.Entry<AffinityKey<Long>, Person>> masters =\n      cache.query(new TextQuery<AffinityKey<Long>, Person>(Person.class, \"Master\"));\n\n    // Query for all people with \"Bachelor Degree\" in their resumes.\n    QueryCursor<Cache.Entry<AffinityKey<Long>, Person>> bachelors =\n      cache.query(new TextQuery<AffinityKey<Long>, Person>(Person.class, \"Bachelor\"));\n\n    print(\"Following people have 'Master Degree' in their resumes: \", masters.getAll());\n    print(\"Following people have 'Bachelor Degree' in their resumes: \", bachelors.getAll());\n  }\n\n  /**\n     * Example for SQL queries to calculate average salary for a specific organization.\n     */\n  private static void sqlQueryWithAggregation() {\n    IgniteCache<AffinityKey<Long>, Person> cache = Ignition.ignite().cache(COLLOCATED_PERSON_CACHE);\n\n    // Calculate average of salary of all persons in ApacheIgnite.\n    // Note that we also join on Organization cache as well.\n    String sql =\n      \"select avg(salary) \" +\n      \"from Person, \\\"\" + ORG_CACHE + \"\\\".Organization as org \" +\n      \"where Person.orgId = org.id \" +\n      \"and lower(org.name) = lower(?)\";\n\n    QueryCursor<List<?>> cursor = cache.query(new SqlFieldsQuery(sql).setArgs(\"ApacheIgnite\"));\n\n    // Calculate average salary for a specific organization.\n    print(\"Average salary for 'ApacheIgnite' employees: \", cursor.getAll());\n  }\n\n  /**\n     * Example for SQL-based fields queries that return only required\n     * fields instead of whole key-value pairs.\n     */\n  private static void sqlFieldsQuery() {\n    IgniteCache<AffinityKey<Long>, Person> cache = Ignition.ignite().cache(PERSON_CACHE);\n\n    // Execute query to get names of all employees.\n    QueryCursor<List<?>> cursor = cache.query(new SqlFieldsQuery(\n      \"select concat(firstName, ' ', lastName) from Person\"));\n\n    // In this particular case each row will have one element with full name of an employees.\n    List<List<?>> res = cursor.getAll();\n\n    // Print names.\n    print(\"Names of all employees:\", res);\n  }\n\n  /**\n     * Example for SQL-based fields queries that return only required\n     * fields instead of whole key-value pairs.\n     */\n  private static void sqlFieldsQueryWithJoin() {\n    IgniteCache<AffinityKey<Long>, Person> cache = Ignition.ignite().cache(COLLOCATED_PERSON_CACHE);\n\n    // Execute query to get names of all employees.\n    String sql =\n      \"select concat(firstName, ' ', lastName), org.name \" +\n      \"from Person, \\\"\" + ORG_CACHE + \"\\\".Organization as org \" +\n      \"where Person.orgId = org.id\";\n\n    QueryCursor<List<?>> cursor = cache.query(new SqlFieldsQuery(sql));\n\n    // In this particular case each row will have one element with full name of an employees.\n    List<List<?>> res = cursor.getAll();\n\n    // Print persons' names and organizations' names.\n    print(\"Names of all employees and organizations they belong to: \", res);\n  }\n\n  /**\n     * Populate cache with test data.\n     */\n  private static void initialize() {\n    IgniteCache<Long, Organization> orgCache = Ignition.ignite().cache(ORG_CACHE);\n\n    // Clear cache before running the example.\n    orgCache.clear();\n\n    // Organizations.\n    Organization org1 = new Organization(\"ApacheIgnite\");\n    Organization org2 = new Organization(\"Other\");\n\n    orgCache.put(org1.id(), org1);\n    orgCache.put(org2.id(), org2);\n\n    IgniteCache<AffinityKey<Long>, Person> colPersonCache = Ignition.ignite().cache(COLLOCATED_PERSON_CACHE);\n    IgniteCache<Long, Person> personCache = Ignition.ignite().cache(PERSON_CACHE);\n\n    // Clear caches before running the example.\n    colPersonCache.clear();\n    personCache.clear();\n\n    // People.\n    Person p1 = new Person(org1, \"John\", \"Doe\", 2000, \"John Doe has Master Degree.\");\n    Person p2 = new Person(org1, \"Jane\", \"Doe\", 1000, \"Jane Doe has Bachelor Degree.\");\n    Person p3 = new Person(org2, \"John\", \"Smith\", 1000, \"John Smith has Bachelor Degree.\");\n    Person p4 = new Person(org2, \"Jane\", \"Smith\", 2000, \"Jane Smith has Master Degree.\");\n\n    // Note that in this example we use custom affinity key for Person objects\n    // to ensure that all persons are collocated with their organizations.\n    colPersonCache.put(p1.key(), p1);\n    colPersonCache.put(p2.key(), p2);\n    colPersonCache.put(p3.key(), p3);\n    colPersonCache.put(p4.key(), p4);\n\n    // These Person objects are not collocated with their organizations.\n    personCache.put(p1.id, p1);\n    personCache.put(p2.id, p2);\n    personCache.put(p3.id, p3);\n    personCache.put(p4.id, p4);\n  }\n\n  /**\n     * Prints message and query results.\n     *\n     * @param msg Message to print before all objects are printed.\n     * @param col Query results.\n     */\n  private static void print(String msg, Iterable<?> col) {\n    print(msg);\n    print(col);\n  }\n\n  /**\n     * Prints message.\n     *\n     * @param msg Message to print before all objects are printed.\n     */\n  private static void print(String msg) {\n    System.out.println();\n    System.out.println(\">>> \" + msg);\n  }\n\n  /**\n     * Prints query results.\n     *\n     * @param col Query results.\n     */\n  private static void print(Iterable<?> col) {\n    for (Object next : col)\n      System.out.println(\">>>     \" + next);\n  }\n}\n",
      "language": "java",
      "name": "Example"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Connecting From PHP to Ignite Cluster"
}
[/block]
TBD: more details to be added

You also are going to need properly configured DSN for Ignite. In the example below we assume that your DSN's name is "Apache Ignite DSN".
[block:callout]
{
  "type": "info",
  "title": "Note that you can't connect with PDO ODBC without properly configured DSN."
}
[/block]
Once you have all these things, you can finally write some code using PDO to connect to Apache Ignite node.
[block:code]
{
  "codes": [
    {
      "code": "<?php\ntry {\n  $dbh = new PDO('odbc:Apache Ignite DSN');\n  \n} catch (PDOException $e) {\n  print \"Error!: \" . $e->getMessage() . \"\\n\";\n  die();\n}\n?>",
      "language": "php"
    }
  ]
}
[/block]
All done. Now you can use your PDO connection with Apache Ignite as any other PDO connection.