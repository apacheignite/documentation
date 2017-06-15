Apache Ignite 2.1 release includes implementation of Ordinary least squares multiple linear regression which is one of most basic ML tools. Multiple linear regression can be described as matrix equation like following:

y=X*b+u.

The term 'X' is a (n,m) matrix, the term 'b' is a k-vector called 'regression parameters' and the term 'u' is a k-vector called 'residuals'. The prediction of this model will be the following:


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
      "code": "regression.newSampleData(new DenseLocalOnHeapVector(y), new DenseLocalOnHeapMatrix(x));",
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