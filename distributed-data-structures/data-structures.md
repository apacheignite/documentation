Ignite allows for most of the data structures from java.util.concurrent framework to be used in a distributed fashion.

Ignite gives you the capability to take a data structure you are familiar with and use it in a clustered fashion. For example, you can take java.util.concurrent.BlockingDeque and add something to it on one node and poll it from another node. Or have a distributed primary key generator which would guarantee uniqueness on all nodes.

Currently you can find the following distributed data structures in Ignite:
* [Queue and Set](doc:queue-and-set) 
* [Atomic Types](doc:atomic-types) 
* [CountDownLatch](doc:countdownlatch) 
* [ID Generator](doc:id-generator) 
* [Semaphore](doc:distributed-semaphore)