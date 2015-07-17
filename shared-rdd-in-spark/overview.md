Apache Ignite provides an implementation of Spark RDD abstraction which allows to easily share state in memory across Spark jobs. The main difference between native Spark RDD and `IgniteRDD` is that Ignite RDD provides a shared in-memory view on data across different Spark jobs, workers, or applications, while native Spark RDD cannot be seen by other Spark jobs or applications.

The way IgniteRDD is implemented is as a view over a distributed Ignite cache, which may be deployed either within the Spark job executing process, or on a Spark worker, or in its own cluster. This means that depending on the chosen deployment mode the shared state may either exist only during the lifespan of a Spark application (embedded mode), or it may out-survive the Spark application (standalone mode) in which case the state can be shared across multiple Spark applications.
[block:image]
{
  "images": [
    {
      "image": [
        "https://www.filepicker.io/api/file/xhEwFkHYRYJAV0PGAmSw",
        "diagram 13_2.png",
        "900",
        "740",
        "#6688cb",
        ""
      ]
    }
  ]
}
[/block]