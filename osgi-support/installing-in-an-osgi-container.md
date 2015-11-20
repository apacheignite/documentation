[block:api-header]
{
  "type": "basic",
  "title": "Setting up the container"
}
[/block]
To start Apache Ignite inside an OSGi container you must have at least installed the following bundles:

* ignite-core
* ignite-osgi
* javax cache API

If you are using Apache Karaf, you may take a shortcut by using the Ignite Feature Repository to install the `ignite-core` feature. See the "Installation on Apache Karaf" section for more information.

You can install additional Ignite modules to expand the platform's functionality as you would do by adding modules to the classpath in a standard environment.


[block:api-header]
{
  "type": "basic",
  "title": "Implementing the Ignite Activator"
}
[/block]
To start Apache Ignite, you will need to implement an OSGi Bundle Activator by extending the abstract class `org.apache.ignite.osgi.IgniteOsgiContextActivator`:
[block:code]
{
  "codes": [
    {
      "code": "import org.apache.ignite.configuration.IgniteConfiguration;\nimport org.apache.ignite.osgi.IgniteOsgiContextActivator;\nimport org.apache.ignite.osgi.classloaders.OsgiClassLoadingStrategyType;\n\npublic class MyActivator extends IgniteOsgiContextActivator {\n\n    /**\n     * Configure your Ignite instance as you would normally do, \n     * and return it.\n     */\n    @Override public IgniteConfiguration igniteConfiguration() {\n        IgniteConfiguration config = new IgniteConfiguration();\n        config.setGridName(\"testGrid\");\n      \n        // ...\n\n        return config;\n    }\n\n    /**\n     * Choose the classloading strategy for Ignite to use.\n     */\n    @Override public OsgiClassLoadingStrategyType classLoadingStrategy() {\n        return OsgiClassLoadingStrategyType.BUNDLE_DELEGATING;\n    }\n}",
      "language": "java"
    }
  ]
}
[/block]
We support two different classloading strategies in OSGi:

* `BUNDLE_DELEGATING`: Uses the classloader of the bundle containing the Activator as a first preference, falling back to the classloader of `ignite-core` in second instance.
* `CONTAINER_SWEEP`: Same as `BUNDLE_DELEGATING`, but ultimately enquires all bundles if the class is still not found.