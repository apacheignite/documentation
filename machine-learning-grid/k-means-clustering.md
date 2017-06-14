For now Apache Ignite ML module provides K-Means algorithm in two versions: KMeansLocalClusterer and KMeansDistributedClusterer for clustering local and distributed data correspondingly. 
 
Let’s begin with KMeansLocal clusterer. It is parametrized by distance measure, seed used in random operations and maximal number of iterations:
[block:code]
{
  "codes": [
    {
      "code": "KMeansLocalClusterer clusterer = new KMeansLocalClusterer(new EuclideanDistance(), 1, 1L);",
      "language": "java"
    }
  ]
}
[/block]
Let’s create an input for this clusterer:
[block:code]
{
  "codes": [
    {
      "code": "double[] v1 = new double[] {1959, 325100};\ndouble[] v2 = new double[] {1960, 373200};\n\nDenseLocalOnHeapMatrix points = new DenseLocalOnHeapMatrix(new double[][] {v1, v2});",
      "language": "java"
    }
  ]
}
[/block]
Then we feed this points to the method 'cluster':
[block:code]
{
  "codes": [
    {
      "code": "KMeansModel mdl = clusterer.cluster(points, 1)",
      "language": "java"
    }
  ]
}
[/block]
and get the model which contains information about clustering: we can get centers of clusters:
[block:code]
{
  "codes": [
    {
      "code": "mdl.centers()\n",
      "language": "java"
    }
  ]
}
[/block]
Also we can predict which cluster some given point belongs to:
[block:code]
{
  "codes": [
    {
      "code": "Integer clusterInx = mdl.predict(new DenseLocalOnHeapVector(new double[] {20.0, 30.0}))",
      "language": "java"
    }
  ]
}
[/block]
**KMeansDistributedClusterer** works in the same way, but it requires ignite to be started and accepts SparseLocalOnHeapMatrix as a parameter.
 
Note: for now we require exactly **DenseLocalOnHeap** matrix to be input for KMeansLocalClusterer, but actually any local Matrix will work. So in future when we’ll have separate interface for local Matrices, the method will accept any of them. The same goes for distributed matrices.