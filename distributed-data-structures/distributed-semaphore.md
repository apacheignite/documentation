Ignite's counting distributed semaphore implementation and behavior is similar to the concept of a well-known `java.util.concurrent.Semaphore`. As any other semaphore it maintains a set of permits that are taken using `acquire()` method and released with `release()` counterpart allowing to restrict access to some logical or physical resource or synchronize execution flow. The only difference is that Ignite's semaphore empowers you to fulfill these kind of actions not only in boundaries of a single JVM but rather a cluster wide, across many remote nodes.   

To start using a distributed semaphore it has to be created in a way as in the example below:
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nIgniteSemaphore semaphore = ignite.semaphore(\n    \"semName\", // Distributed semaphore name.\n    20,        // Number of permits.\n    true,      // Release acquired permits if node, that owned them, left topology.\n    true       // Create if it doesn't exist.\n);",
      "language": "java"
    }
  ]
}
[/block]