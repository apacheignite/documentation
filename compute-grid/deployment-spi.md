[block:api-header]
{
  "type": "basic",
  "title": "DeploymentSpi"
}
[/block]
Deployment functionality is provided via `DeploymentSpi` interface.

Class loaders that are in charge of loading task classes (and other classes) can be deployed directly by calling `register(ClassLoader, Class)` method or by SPI itself, for example by asynchronously scanning some folder for new tasks. When method `findResource(String)` is called by the system, SPI must return a class loader associated with given class. Every time a class loader gets (re)deployed or released, callbacks `DeploymentListener.onUnregistered(ClassLoader)`} must be called by SPI.

If peer class loading is enabled, then it is usually enough to deploy class loader only on one grid node. Once a task starts executing on the grid, all other nodes will automatically load all task classes from the node that initiated the execution. Hot redeployment is also supported with peer class loading. Every time a task changes and gets redeployed on a node, all other nodes will detect it and will redeploy this task as well. Note that peer class loading comes into effect only if a task was not locally deployed, otherwise, preference will always be given to local deployment.

Ignite provides these following `DeploymentSpi` implementations out of the box:
  * UriDeploymentSpi
  * LocalDeploymentSpi
[block:callout]
{
  "type": "warning",
  "body": "SPI methods should never be used directly. SPIs provide internal view on the subsystem and is used internally by Ignite. In rare use cases when access to a specific implementation of this SPI is required - an instance of this SPI can be obtained via Ignite.configuration() method to check its configuration properties or call other non-SPI methods. Note again that calling methods from this interface on the obtained instance can lead to undefined behavior and explicitly not supported."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "UriDeploymentSpi"
}
[/block]
This is the implementation of `DeploymentSpi` which can deploy tasks from different sources like file system folders, email and HTTP. There are different ways to deploy tasks in grid and every deploy method depends on selected source protocol. This SPI is configured to work with a list of URI's. Every URI contains all the data about protocol/transport plus configuration parameters like credentials, scan frequency, and others.

When SPI establishes a connection with an URI, it downloads deployable units to the temporary directory in order to prevent it from any changes while scanning. Use method `setTemporaryDirectoryPath(String)` to set custom temporary folder for downloaded deployment units. SPI will create folder under the path with name identical to local node ID.

SPI tracks all changes of every given URI. This means that if any file is changed or deleted, SPI will re-deploy or delete corresponding tasks. Note that the very first apply to `findResource(String)` is blocked until SPI finishes scanning all URI's at least once.

There are several deployable unit types supported:
  * GAR file.
  * Local disk folder with structure of unpacked GAR file.
  * Local disk folder containing only compiled Java classes.

## GAR file
GAR file is a deployable unit. GAR file is based on ZLIB compression format like simple JAR file and its structure is similar to WAR archive. GAR file has '.gar' extension.

GAR file structure (file or directory ending with '.gar'):
[block:code]
{
  "codes": [
    {
      "code": "META-INF/\n        |\n         - ignite.xml\n         - ...\nlib/\n   |\n    -some-lib.jar\n    - ...\nxyz.class\n...",
      "language": "text"
    }
  ]
}
[/block]
  * `META-INF/` entry may contain `ignite.xml` file which is a task descriptor file. The purpose of task descriptor XML file is to specify all tasks to be deployed. This file is a regular Spring XML definition file. `META-INF/` may also contain any other files specific for JAR format.
  * `lib/` entry contains all library dependencies.
  * Compiled Java classes must be placed in the root of a GAR file.

GAR file may be deployed without descriptor file. If there is no descriptor file, SPI will scan all classes in archive and instantiate those that implement `ComputeTask` interface. In that case, all grid task classes must have a public no-argument constructor. Use `ComputeTaskAdapter` adapter for convenience when creating grid tasks.

By default, all downloaded GAR files that have digital signature in `META-INF` folder will be verified and deployed only if signature is valid.

## Code Example
The following example demonstrates how the deployment SPI can be used. It expects that you have a GAR file in `home/username/ignite/work/my_deployment/file` folder which contains `myproject.HelloWorldTask` class.
[block:code]
{
  "codes": [
    {
      "code": "IgniteConfiguration cfg = new IgniteConfiguration();\n\nDeploymentSpi deploymentSpi = new UriDeploymentSpi();\n\ndeploymentSpi.setUriList(Arrays.asList(\"file:///home/username/ignite/work/my_deployment/file\"));\n\ncfg.setDeploymentSpi(deploymentSpi);\n\ntry(Ignite ignite = Ignition.start(cfg)) {\n\t  ignite.compute().execute(\"myproject.HelloWorldTask\", \"my args\");\n}",
      "language": "java"
    }
  ]
}
[/block]
## Configuration
[block:parameters]
{
  "data": {
    "h-0": "Property",
    "h-1": "Description",
    "h-2": "Optional",
    "h-3": "Default",
    "0-0": "`uriList`",
    "0-1": "List of URIs that should be scanned by SPI for the new tasks.",
    "0-2": "Yes",
    "0-3": "`file://${IGNITE_HOME}/work/deployment/file` (note that `IGNITE_HOME` must be set if you're using the default folder).",
    "1-0": "`scanners`",
    "1-1": "Array of `UriDeploymentScanner` implementations which will be used to deploy resources.",
    "1-2": "Yes",
    "1-3": "`UriDeploymentFileScanner` and `UriDeploymentHttpScanner`.",
    "2-0": "`temporaryDirectoryPath`",
    "2-1": "Temporary directory path where scanned GAR files and directories are copied to.",
    "2-2": "Yes",
    "2-3": "`java.io.tmpdir` system property value.",
    "3-0": "`encodeUri`",
    "3-1": "Flag to control encoding of the `path` portion of URI.",
    "3-2": "Yes",
    "3-3": "`true`"
  },
  "cols": 4,
  "rows": 4
}
[/block]
## Protocols
Following protocols are supported in SPI out of the box:
  * file:// - File protocol
  * http:// - HTTP protocol
  * https:// - Secure HTTP protocol
[block:callout]
{
  "type": "info",
  "title": "Custom Protocols",
  "body": "You can add support for additional protocols if needed. To do this implement `UriDeploymentScanner` interface and plug your implementation into the SPI via `setScanners(UriDeploymentScanner... scanners)` method."
}
[/block]
In addition to SPI configuration parameters, all necessary configuration parameters for selected URI should be defined in URI. Different protocols have different configuration parameters described below. Parameters are separated by ';' character.

### File
For this protocol SPI will scan folder specified by URI on file system and download any GAR files or directories that end with .gar from source directory defined in URI. For file system URI must have scheme equal to file.

Following parameters are supported:
[block:parameters]
{
  "data": {
    "h-0": "Parameter",
    "h-1": "Description",
    "h-2": "Optional",
    "h-3": "Default",
    "0-0": "`freq`",
    "0-1": "Scanning frequency in milliseconds.",
    "0-2": "Yes",
    "0-3": "`5000` ms"
  },
  "cols": 4,
  "rows": 1
}
[/block]
### File Uri Example
The following example will scan `c:/Program files/ignite/deployment` folder on local box every '2000' milliseconds. Note that since path has spaces, `setEncodeUri(boolean)` parameter must be set to true (which is default behavior).
[block:code]
{
  "codes": [
    {
      "code": "file://freq=2000@localhost/c:/Program files/ignite/deployment",
      "language": "text"
    }
  ]
}
[/block]
### HTTP/HTTPS
URI deployment scanner tries to read the DOM of the HTML file it points to and parses out href attributes of all &lt;a&gt;-tags - this becomes the URL collection (to GAR files) to deploy. It's important that only HTTP scanner uses URLConnection.getLastModified() method to check if there were any changes since last iteration for each GAR-file before redeploying. 

Following parameters are supported:
[block:parameters]
{
  "data": {
    "h-0": "Parameter",
    "h-1": "Description",
    "h-2": "Optional",
    "h-3": "Default",
    "0-0": "`freq`",
    "0-1": "Scanning frequency in milliseconds.",
    "0-2": "Yes",
    "0-3": "`300000` ms"
  },
  "cols": 4,
  "rows": 1
}
[/block]
### Http Uri Example
The following example will scan `ignite/deployment` folder on site `www.mysite.com` using authentication 'username:password' every '10000' milliseconds.
[block:code]
{
  "codes": [
    {
      "code": "http://username:password;freq=10000@www.mysite.com:110/ignite/deployment",
      "language": "text"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "LocalDeploymentSpi"
}
[/block]
Local deployment SPI that implements only within VM deployment on local node via `register(ClassLoader, Class)` method. This SPI requires no configuration.

There is no point to explicitly configure `LocalDeploymentSpi` as it is used by default and has no configuration parameters.