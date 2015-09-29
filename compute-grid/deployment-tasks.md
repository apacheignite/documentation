[block:api-header]
{
  "type": "basic",
  "title": "DeploymentSpi"
}
[/block]
Deployment functionality is provided via `DeploymentSpi` interface.

Class loaders that are in charge of loading task classes (and other classes) can be deployed directly by calling `register(ClassLoader, Class)` method or by SPI itself, for example by asynchronously scanning some folder for new tasks. When method `findResource(String)` is called by the system, SPI must return a class loader associated with given class. Every time a class loader gets (re)deployed or released, callbacks `DeploymentListener.onUnregistered(ClassLoader)`} must be called by SPI.

If peer class loading is enabled (which is default behavior, see `IgniteConfiguration.isPeerClassLoadingEnabled()`), then it is usually enough to deploy class loader only on one grid node. Once a task starts executing on the grid, all other nodes will automatically load all task classes from the node that initiated the execution. Hot redeployment is also supported with peer class loading. Every time a task changes and gets redeployed on a node, all other nodes will detect it and will redeploy this task as well. Note that peer class loading comes into effect only if a task was not locally deployed, otherwise, preference will always be given to local deployment.

Ignite provides the following GridDeploymentSpi implementations:
  * LocalDeploymentSpi[](http://apacheignite.gridgain.org/v1.4/docs/deployment-tasks#urideploymentspi)
  * UriDeploymentSpi

NOTE: this SPI (i.e. methods in this interface) should never be used directly. SPIs provide internal view on the subsystem and is used internally by Ignite kernal. In rare use cases when access to a specific implementation of this SPI is required - an instance of this SPI can be obtained via Ignite.configuration() method to check its configuration properties or call other non-SPI methods. Note again that calling methods from this interface on the obtained instance can lead to undefined behavior and explicitly not supported.
[block:api-header]
{
  "type": "basic",
  "title": "UriDeploymentSpi"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Local"
}
[/block]