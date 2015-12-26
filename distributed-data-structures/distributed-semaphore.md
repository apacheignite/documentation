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
Once the semaphore is created it can be concurrently used by multiple nodes of the cluster in order to implement some distributed logic or restrict access to a distributed resource like in the following example:
[block:code]
{
  "codes": [
    {
      "code": "Ignite ignite = Ignition.ignite();\n\nIgniteSemaphore semaphore = ignite.semaphore(\n    \"semName\", // Distributed semaphore name.\n    20,        // Number of permits.\n    true,      // Release acquired permits if node, that owned them, left topology.\n    true       // Create if it doesn't exist.\n);\n\n// Acquires a permit, blocking until it's available.\nsemaphore.acquire();\n\ntry {\n    // Semaphore permit is acquired. Execute a distributed task.\n    ignite.compute().run(() -> {\n        System.out.println(\"Executed on:\" + ignite.cluster().localNode().id());\n  \n        // Additional logic.\n    });\n}\nfinally {\n    // Releases a permit, returning it to the semaphore.\n    semaphore.release();\n}",
      "language": "java"
    }
  ]
}
[/block]