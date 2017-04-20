[block:api-header]
{
  "title": "Overview"
}
[/block]
Apache Ignite 2.0 release introduced a beta version of Apache Ignite Machine Learning Grid (ML Grid) which is a distributed machine learning library built on top of Apache Ignite Data Fabric.

Presently, the beta version supports the following functionality:
*  An ability to perform vector and matrix algebra operations including local and distributed, dense and sparse logic.
[block:callout]
{
  "type": "success",
  "title": "ML Grid Roadmap",
  "body": "In the future, ML Grid will be empowered with distributed versions of the well-known algorithms used for machine learning tasks and predictive analysis. In addition, ML Grid API will be available for such programming languages as Python and Ruby."
}
[/block]
This guide walks you through how to build and get started using Ignite ML.


Ignite ML is provided as source release. It requires Java 8.

As of now package

Readers who used ML related features in libraries like Apache Mahout and Colt (https://en.wikipedia.org/wiki/Colt_(libraries)) will find the API generally familiar. Ignite ML API was designed to make it easier for those already working with ML matters get used to.

ML package can be obtained from Ignite project root in a module called “ml” (“ignite-ml”), package org.apache.ignite.ml

[block:api-header]
{
  "title": "Getting Started"
}
[/block]
The fast way to get started with Ignite ML is to build and run the examples, study their output and code. ML examples are located in examples module under ignite root, in package org.apache.ignite.examples.ml

Running examples does not require any special configuration. All Ignite ML examples are supposed to launch, run and stop successfully without any user intervention and provide meaningful output into console.

Example for Tracer API is additionally supposed to launch a web browser and do some HTML output into browser window.


#Build details


If needed, refer DEVNOTES.txt file in project root and readme files in ignite-ml and examples modules for more details.

The way used for command line build check in a local copy of ignite-2.0 branch commit [9e7421f] is as follows:

1. clean local Maven repo
   (this is to ensure that older Maven builds don’t impact my check)

2. build and local install all ignite from project root directory
  mvn clean install -DskipTests -Dmaven.javadoc.skip=true -P java8

3. build and local install ML from project root directory
  mvn install -Pml -DskipTests -U -pl modules/ml -am

4. build examples
  cd examples
  mvn clean package -DskipTests -Pml

The same codebase was used to verify that Ignite ML unit tests and examples run as documented in this guide.


#API usage reference in unit tests


The most reliable way to learn how particular Ignite ML API is intended to be used is to find how this is done in respective unit tests. Unit tests were specifically designed with this use in mind and they cover vast majority of the package API.

Running unit tests does not require any special configuration. All unit tests in Ignite ML package are supposed to launch and pass successfully without any user intervention.

Some unit tests for Tracer API are additionally supposed to launch a web browser and do some HTML output into browser window.

In order to execute all unit tests except for TracerTest launch MathImplMainTestSuite.

You can also refer module javadocs for the explanation of the API packages, classes, and methods.


#References


JIRA: https://issues.apache.org/jira/browse/IGNITE-4572
    Machine Learning: Develop distributed algebra support for dense and sparse data sets.

Dev list: http://apache-ignite-developers.2346864.n4.nabble.com/Adding-ML-to-Ignite-IGNITE-4572-tp13936.html
    Adding ML to Ignite, IGNITE-4572