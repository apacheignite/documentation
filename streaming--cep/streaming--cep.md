Ignite streaming allows to process continuous never-ending streams of data in scalable and fault-tolerant fashion. The rates at which data can be injected into Ignite can be very high and easily exceed millions of events per second on a moderately sized cluster. 

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
##Data Streamers
Data streamers are defined by `IgniteDataStreamer` API and are built to inject large amounts of continuous streams of data into Ignite stream caches. Data streamers are built in a scalable and fault-tolerant fashion and provide **at-least-once-guarantee** semantics for all the data streamed into Ignite.

[read more](doc:data-streamers)

## Sliding Windows
Sliding windows are configured as Ignite cache eviction policies, and can be time-based, size-based, or batch-based. You can configure one sliding-window per cache. However, you can easily define more than one cache if you need different sliding windows for the same data.

[read more](doc:sliding-windows) 

##Querying Data
You can use Ignite full set of Ignite data indexing capabilities, together with Ignite SQL, TEXT, and Predicate based cache queries, to query into the streaming data.

[read more](doc:cache-queries)