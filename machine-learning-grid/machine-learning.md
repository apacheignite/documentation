[block:api-header]
{
  "title": "Overview"
}
[/block]
Apache Ignite 2.0 release introduced a beta version of Apache Ignite Machine Learning Grid (ML Grid) which is a distributed machine learning library built on top of highly optimized and scalable Apache Ignite Data Fabric.

Those who use (or have used) ML related features in libraries like [Apache Mahout](http://mahout.apache.org) and [Colt](https://en.wikipedia.org/wiki/Colt_(libraries) will find the API familiar. Apache Ignite ML Grid API was designed to make it easier for those already working with ML matters get used to it.

Presently, the beta version supports the following functionality:
*  Ability to perform vector and matrix algebra operations including local and distributed, dense and sparse logic.
[block:callout]
{
  "type": "success",
  "title": "ML Grid Roadmap",
  "body": "In later releases, ML Grid will be empowered with distributed versions of the well-known algorithms used for machine learning tasks and predictive analysis. In addition, ML Grid API will be available for programming languages such as Python and Ruby."
}
[/block]

[block:api-header]
{
  "title": "Getting Started"
}
[/block]
The fastest way to get started with the ML Grid is to build and run the examples, study their output and code. ML examples are located in the `examples` folder of the Apache Ignite distribution. Here is a [direct GitHub link](https://github.com/apache/ignite/tree/master/examples/src/main/ml/org/apache/ignite/examples/ml/math) to them.

Follow the steps below to try out the examples:
1. Make sure you're using Java 8 or later. 
2. Download Apache Ignite of version 2.0 or later.
3. Open `examples` project in an IDE like IntelliJ IDEA or Eclipse.
4. Activate `ml` Maven profile when setting up the project.
5. Go to `src\main\ml` folder in the IDE and run an ML Grid example.
 
The examples do not require any special configuration. All ML Grid examples are supposed to launch, run and stop successfully without any user intervention and provide meaningful output on the console. Additionally, an example for the Tracer API is supposed to launch a web browser and provide some HTML output.
[block:api-header]
{
  "title": "Build From Sources"
}
[/block]
The latest Apache Ignite ML Grid jar is uploaded to the Maven repository. If you need to take the jar and deploy it in a custom environment, then it can be either downloaded from Maven or built from scratch. To build ML Grid from sources:
1. Download the latest Apache Ignite source release.
2. Clean local Maven repo (this is to ensure that older Maven builds don’t impact my check).
3. Make sure you're using Java 8 or later. 
4. Build and install Apache Ignite Data Fabric from the project's root directory:
[block:code]
{
  "codes": [
    {
      "code": "mvn clean install -DskipTests -Dmaven.javadoc.skip=true -P java8",
      "language": "shell"
    }
  ]
}
[/block]
5. Build and install ML Grid from the project's root directory:
[block:code]
{
  "codes": [
    {
      "code": "  mvn install -Pml -DskipTests -U -pl modules/ml -am",
      "language": "shell"
    }
  ]
}
[/block]
6. Locate the ML Grid jar in your local Maven repository under the path `{user_dir}/.m2/repository/org/apache/ignite/ignite-ml/{ignite-version}/ignite-ml-{ignite-version}.jar`.

If needed, refer to `DEVNOTES.txt` in the project's root folder and `README` files in the `ignite-ml` component for more details.