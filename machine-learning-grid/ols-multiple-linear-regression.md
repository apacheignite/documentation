Apache Ignite 2.1 release includes implementation of Ordinary least squares multiple linear regression which is one of most basic ML tools. Multiple linear regression parameters (b, u) can be described as a solution of the following equation

y = X*b + u, 

minimizing sum of squares of u. Here **X** is a (**n**,**m**) matrix, the term **b** is a k-vector called 'regression parameters' and the term **u** is a **k**-vector called 'residuals'. Geometrically solutions of equation above are lines in an (**m+1**)-dimensional vector space.

Matrix **X** should be considered as an input on which the model trains. 

Below follows the description of it's usage.

First, we should create a new instance of OLSMultipleLinearRegression class:
[block:code]
{
  "codes": [
    {
      "code": "OLSMultipleLinearRegression regression = new OLSMultipleLinearRegression();",
      "language": "java"
    }
  ]
}
[/block]
Next we'll need some data.
[block:code]
{
  "codes": [
    {
      "code": "y = new double[] {11.0, 12.0, 13.0, 14.0, 15.0, 16.0};\nx = new double[6][];\nx[0] = new double[] {0, 0, 0, 0, 0};\nx[1] = new double[] {2.0, 0, 0, 0, 0};\nx[2] = new double[] {0, 3.0, 0, 0, 0};\nx[3] = new double[] {0, 0, 4.0, 0, 0};\nx[4] = new double[] {0, 0, 0, 5.0, 0};\nx[5] = new double[] {0, 0, 0, 0, 6.0};",
      "language": "java"
    }
  ]
}
[/block]
Let's load it into our regression.
[block:code]
{
  "codes": [
    {
      "code": "DenseLocalOnHeapMatrix xm = new DenseLocalOnHeapMatrix(m).\n\nregression.newSampleData(new DenseLocalOnHeapVector(y), m);",
      "language": "java"
    }
  ]
}
[/block]
By default, OLSMultipleRegression augments input matrix with an additional column inserted from the left filled with 1s. This is done for model to be able to give lines passing not through the origin, but if we want, we can disable this augmenting with
[block:code]
{
  "codes": [
    {
      "code": "model.setNoIntercept(true);",
      "language": "java"
    }
  ]
}
[/block]
Now we can estimate regression parameters:
[block:code]
{
  "codes": [
    {
      "code": "double[] betaHat = regression.estimateRegressionParameters();",
      "language": "java"
    }
  ]
}
[/block]
and residuals
[block:code]
{
  "codes": [
    {
      "code": "double[] residuals = regression.estimateResiduals();",
      "language": "java"
    }
  ]
}
[/block]
**Note**, that the current API of Apache Ignite ML module is the subject of change.