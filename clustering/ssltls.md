* [Securing Connection Between Nodes](#securing-connection-between-nodes)
* [SSL and TLS](#ssl-and-tls)
* [Configuration](#configuration)
[block:api-header]
{
  "type": "basic",
  "title": "Securing Connection Between Nodes"
}
[/block]
Ignite allows you to use SSL socket communication to provide a secure connection among all Ignite nodes. To use it, set the `Factory<SSLContext>` and configure the SSL section in the Ignite configuration. Ignite provides a default SSL context factory, `org.apache.ignite.ssl.SslContextFactory`, which uses a configurable keystore to initialize the SSL context. 
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  <property name=\"sslContextFactory\">\n    <bean class=\"org.apache.ignite.ssl.SslContextFactory\">\n      <property name=\"keyStoreFilePath\" value=\"keystore/server.jks\"/>\n      <property name=\"keyStorePassword\" value=\"123456\"/>\n      <property name=\"trustStoreFilePath\" value=\"keystore/trust.jks\"/>\n      <property name=\"trustStorePassword\" value=\"123456\"/>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration igniteCfg = new IgniteConfiguration();\n\nSslContextFactory factory = new SslContextFactory();\n\nfactory.setKeyStoreFilePath(\"keystore/server.jks\");\nfactory.setKeyStorePassword(\"123456\".toCharArray());\nfactory.setTrustStoreFilePath(\"keystore/trust.jks\");\nfactory.setTrustStorePassword(\"123456\".toCharArray());\n\nigniteCfg.setSslContextFactory(factory);",
      "language": "java"
    }
  ]
}
[/block]
In some cases, it is useful to disable certificate validation on the client side, such as when connecting to a server with a self-signed certificate. This can be achieved by setting a disabled trust manager to this factory, which can be obtained by the `getDisabledTrustManager` method.
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  <property name=\"sslContextFactory\">\n    <bean class=\"org.apache.ignite.ssl.SslContextFactory\">\n      <property name=\"keyStoreFilePath\" value=\"keystore/server.jks\"/>\n      <property name=\"keyStorePassword\" value=\"123456\"/>\n      <property name=\"trustManagers\">\n        <bean class=\"org.apache.ignite.ssl.SslContextFactory\" factory-method=\"getDisabledTrustManager\"/>\n     </property>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration igniteCfg = new IgniteConfiguration();\n\nSslContextFactory factory = new SslContextFactory();\n\nfactory.setKeyStoreFilePath(\"keystore/server.jks\");\nfactory.setKeyStorePassword(\"123456\".toCharArray());\nfactory.setTrustManagers(SslContextFactory.getDisabledTrustManager());\n\nigniteCfg.setSslContextFactory(factory);",
      "language": "java"
    }
  ]
}
[/block]
If security is configured, then the logs will include `communication encrypted=on`
[block:code]
{
  "codes": [
    {
      "code": "INFO: Security status [authentication=off, communication encrypted=on]",
      "language": "text"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "SSL and TLS"
}
[/block]
Ignite allows the use of different encryption types.  The following algorithms are supported [http://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html#SSLContext](http://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html#SSLContext) and can be set by using the `setProtocol` method. `TLS` encryption is the default.
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  <property name=\"sslContextFactory\">\n    <bean class=\"org.apache.ignite.ssl.SslContextFactory\">\n      <property name=\"setProtocol\" value=\"SSL\"/>\n      ...\n    </bean>\n  </property>\n  ...\n</bean>",
      "language": "xml"
    },
    {
      "code": "IgniteConfiguration igniteCfg = new IgniteConfiguration();\n\nSslContextFactory factory = new SslContextFactory();\n\n...\n  \nfactory.setProtocol(\"TLS\");\n\nigniteCfg.setSslContextFactory(factory);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuration"
}
[/block]
The following configuration parameters can be configured on `SslContextFactory`.
[block:parameters]
{
  "data": {
    "h-0": "Setter Method",
    "h-1": "Description",
    "h-2": "Default",
    "0-0": "`setKeyAlgorithm`",
    "0-1": "Sets the key manager algorithm that will be used to create a key manager. Notice that in most cases the default value work well.  However, on the Android platform, this value need to be set to `X509`.",
    "0-2": "`SunX509`",
    "1-0": "`setKeyStoreFilePath`",
    "1-1": "Sets the path to the key store file. This is a mandatory parameter since the SSL context can not be initialized without a key manager.",
    "1-2": "`N/A`",
    "2-0": "`setKeyStorePassword`",
    "2-1": "Sets the key store password.",
    "2-2": "`N/A`",
    "3-0": "`setKeyStoreType`",
    "3-1": "Sets the key store type used in context initialization.",
    "3-2": "`JKS`",
    "4-0": "`setProtocol`",
    "4-1": "Sets the protocol for secure transport.",
    "4-2": "`TLS`",
    "5-0": "`setTrustStoreFilePath`",
    "5-1": "Sets the path to the trust store file.",
    "5-2": "`N/A`",
    "6-0": "`setTrustStorePassword`",
    "6-1": "Sets the trust store password.",
    "6-2": "`N/A`",
    "7-0": "`setTrustStoreType`",
    "7-1": "Sets the trust store type used in context initialization.",
    "7-2": "`JKS`",
    "8-0": "`setTrustManagers`",
    "8-1": "Sets the pre-configured trust managers.",
    "8-2": "'N/A`"
  },
  "cols": 3,
  "rows": 9
}
[/block]