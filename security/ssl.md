Ignite allows you to use SSL socket communication among all Ignite nodes. To use it, you need to set `Factory<SSLContext>` and configure the SSL section in Ignite configuration. Ignite provides a default SSL context factory, `org.apache.ignite.ssl.SslContextFactory`, which uses configured keystore to initialize SSL context. 
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n  <property name=\"sslContextFactory\">\n    <bean class=\"org.apache.ignite.ssl.SslContextFactory\">\n      <property name=\"keyStoreFilePath\" value=\"keystore/server.jks\"/>\n      <property name=\"keyStorePassword\" value=\"123456\"/>\n      <property name=\"trustStoreFilePath\" value=\"keystore/trust.jks\"/>\n      <property name=\"trustStorePassword\" value=\"123456\"/>\n    </bean>\n  </property>\n</bean>",
      "language": "xml"
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
Following configuration parameters can be optionally configured on `SslContextFactory`.
[block:parameters]
{
  "data": {
    "h-0": "Setter Method",
    "h-1": "Description",
    "h-2": "Default",
    "0-0": "`setKeyAlgorithm`",
    "0-1": "Sets key manager algorithm that will be used to create a key manager. Notice that in most cased default value suites well, however, on Android platform this value need to be set to `X509`.",
    "0-2": "`SunX509`",
    "1-0": "`setKeyStoreFilePath`",
    "1-1": "Sets path to the key store file. This is a mandatory parameter since ssl context could not be initialized without key manager.",
    "1-2": "`N/A`",
    "2-1": "Sets key store password.",
    "2-0": "`setKeyStorePassword`",
    "2-2": "`N/A`",
    "3-1": "Sets key store type used in context initialization.",
    "3-0": "`setKeyStoreType`",
    "3-2": "`JKS`",
    "4-1": "Sets protocol for secure transport.",
    "4-0": "`setProtocol`",
    "4-2": "`TLS`",
    "5-1": "Sets path to the trust store file.",
    "5-2": "`N/A`",
    "5-0": "`setTrustStoreFilePath`",
    "6-0": "`setTrustStorePassword`",
    "6-1": "Sets trust store password.",
    "6-2": "`N/A`",
    "7-0": "`setTrustStoreType`",
    "7-1": "Sets trust store type used in context initialization.",
    "7-2": "`JKS`"
  },
  "cols": 3,
  "rows": 8
}
[/block]