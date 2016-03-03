Ignite streaming allows to process continuous never-ending streams of data in scalable and fault-tolerant fashion. The rates at which data can be injected into Ignite can be very high and easily exceed millions of events per second on a moderately sized cluster. 

##How it Works
  1. Client nodes inject finite or continuous streams of data into Ignite caches using Ignite [Data Streamers](doc:data-streamers). 
  2. Data is automatically partitioned between Ignite data nodes, and each node gets equal amount of data.
  3. Streamed data can be concurrently processed directly on the Ignite data nodes in collocated fashion.
  4. Clients can also perform concurrent SQL queries on the streamed data.
  
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/JWb9VJReTWSUw2PSA2OE",
        "ignite-stream-query.png",
        "600",
        "316",
        "#98785c",
        ""
      ]
    }
  ]
}
[/block]
##Data Streamers
Data streamers are defined by `IgniteDataStreamer` API and are built to inject large amounts of continuous streams of data into Ignite stream caches. Data streamers are built in a scalable and fault-tolerant fashion and provide **at-least-once-guarantee** semantics for all the data streamed into Ignite.

[Read more](doc:data-streamers)

## Sliding Windows
Ignite streaming functionality also allows to query into **sliding windows** of data. Since streaming data never ends, you rarely want to query the whole data set going back to the very beginning. Instead, you are more interested in questions like “What are the 10 most popular products over last 2 hours?”, or “What is the average product price in a certain category for the past day?”. To achieve this, you need to be able to query into *sliding data windows*.
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/OLzREplOTqWuV62KOSrm",
        "in_memory_streaming.png",
        "600",
        "180",
        "#e45e6d",
        ""
      ]
    }
  ]
}
[/block]
Sliding windows are configured as Ignite cache eviction policies, and can be time-based, size-based, or batch-based. You can configure one sliding-window per cache. However, you can easily define more than one cache if you need different sliding windows for the same data.

[Read more](doc:sliding-windows) 

##Querying Data
You can use full set of Ignite data indexing capabilities, together with Ignite SQL, TEXT, and Predicate based cache queries, to query into the streaming data.

[Read more](doc:cache-queries)

##Word Count Example
In this example we will stream text into Ignite and count each individual word. We will also issue periodic SQL queries into the stream to query top 10 most popular words.

[Read More](doc:streaming-example)

##JMS Data Streamer
Ignite offers a JMS Data Streamer to consume messages from JMS brokers, convert them into cache tuples and insert them in Ignite caches.

[Read More](doc:jms-data-streamer)

## Flume Data Streamer
IgniteSink is a Flume sink that extracts Events from an associated Flume channel and injects into an Ignite cache. Flume 1.6.0 is supported.

[Read More](doc:flume-data-streamer)