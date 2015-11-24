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

When deploying in Apache Karaf, you may take a shortcut by using the Ignite Feature Repository to install the `ignite-core` feature. See the "Installation on Apache Karaf" section for more information.

Feel free to install additional Ignite modules to expand the platform's functionality, as you would do by adding modules to the classpath in a standard environment.
[block:api-header]
{
  "type": "basic",
  "title": "Implementing the Ignite Bundle Activator"
}
[/block]
To start Apache Ignite, implement an OSGi Bundle Activator by extending the abstract class `org.apache.ignite.osgi.IgniteOsgiContextActivator`:
[block:code]
{
  "codes": [
    {
      "code": "package org.apache.ignite.osgi.examples;\n\nimport org.apache.ignite.configuration.IgniteConfiguration;\nimport org.apache.ignite.osgi.IgniteOsgiContextActivator;\nimport org.apache.ignite.osgi.classloaders.OsgiClassLoadingStrategyType;\n\npublic class MyActivator extends IgniteOsgiContextActivator {\n\n    /**\n     * Configure your Ignite instance as you would normally do, \n     * and return it.\n     */\n    @Override public IgniteConfiguration igniteConfiguration() {\n        IgniteConfiguration config = new IgniteConfiguration();\n        config.setGridName(\"testGrid\");\n      \n        // ...\n\n        return config;\n    }\n\n    /**\n     * Choose the classloading strategy for Ignite to use.\n     */\n    @Override public OsgiClassLoadingStrategyType classLoadingStrategy() {\n        return OsgiClassLoadingStrategyType.BUNDLE_DELEGATING;\n    }\n}",
      "language": "java",
      "name": "A simple Ignite Activator"
    }
  ]
}
[/block]
We support two different classloading strategies in OSGi:

* `BUNDLE_DELEGATING`: Uses the classloader of the bundle containing the Activator as a first preference, falling back to the classloader of `ignite-core` in second instance.
* `CONTAINER_SWEEP`: Same as `BUNDLE_DELEGATING`, but ultimately enquires all bundles if the class is still not found.
[block:callout]
{
  "type": "info",
  "title": "Future OSGi classloading strategies",
  "body": "We may consider adding other classloading strategies in upcoming releases, such as using the Service Locator mechanism to locate bundles which voluntarily want to expose packages to Ignite's marshallers via a file similar to jaxb.index in the JAXB specification.\n\nReach out to us in the community (users@ignite.apache.org mailing list) if you'd like to see such options in the future."
}
[/block]
Make sure to add the `Bundle-Activator` OSGi Manifest header to your bundle, in order to instruct the OSGi container to call the Activator on bundle start:
[block:code]
{
  "codes": [
    {
      "code": "Bundle-SymbolicName: test-bundle\nBundle-Activator: org.apache.ignite.osgi.examples.MyActivator\nImport-Package: ...\n[...]",
      "language": "text",
      "name": "Excerpt of OSGi headers including Bundle-Activator"
    }
  ]
}
[/block]
To generate the bundle, including the `Bundle-Activator` OSGi header, we recommend adding the [`maven-bundle-plugin`](https://felix.apache.org/documentation/subprojects/apache-felix-maven-bundle-plugin-bnd.html) to your Maven build, with the appropriate configuration instructions:

[block:code]
{
  "codes": [
    {
      "code": "<plugin>\n  <groupId>org.apache.felix</groupId>\n  <artifactId>maven-bundle-plugin</artifactId>\n  <version>${maven.bundle.plugin.version}</version>\n  <configuration>\n    <Bundle-SymbolicName>...</Bundle-SymbolicName>\n    <Bundle-Activator>org.apache.ignite.osgi.examples.MyActivator</Bundle-Activator>\n    [...]\n  </configuration>\n</plugin>",
      "language": "xml",
      "name": "Maven configuration of maven-bundle-plugin"
    }
  ]
}
[/block]