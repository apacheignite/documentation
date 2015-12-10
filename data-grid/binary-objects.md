[block:api-header]
{
  "type": "basic",
  "title": "Basic Concept"
}
[/block]
Starting from v1.5 Ignite introduced a new concept of storing data in caches named BinaryObjects. There are several advantages provided by the new serialization format:
 * It allows to read an arbitrary field from an object's serialized form without full object deserialization. This ability completely removes the requirement to have cache key and value classes deployed to the server nodes classpath. 
 * It allows to add and remove fields to the objects of the same type. Given that server nodes do not have model classes definitions, this ability allows dynamic object structure change, and even allows multiple clients with different versions of class definitions to co-exist.
 * It allows to construct new objects based on a type name without having class definitions at all, which basically allows dynamic type creation.
[block:callout]
{
  "type": "info",
  "title": "Restrictions",
  "body": "There are several restrictions that are implied by the `BinaryObject` format implementation.\n * Internally Ignite does not write field and type names, but uses lower-case name hash to identify a field or a type. This means that fields or types with the same name hash are not allowed. Even though serialization will not work out-of-the box in the case of hash collision, Ignite provides a way to resolve this collision at the configuration level.\n * For the same reason `BinaryObject` format does not allow same field names on different levels of a class hierarchy.\n * `Externalizable` interface is ignored by default. If `BinaryObject` format is used, `Externalizable` classes will be written the same way as if they were `Serializable`, without `writeExternal()` and `readExternal()` methods. If for some reason this does not work for you, you should implement `Binarylizable` interface for your classes, plug in custom `BinarySerializer` or switch to the `OptimizedMarshaller`"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring Binary Objects"
}
[/block]
In the vast majority of use-cases there is no need to additionally configure binary objects. `BinaryObject` marshaller is enabled by default when no other marshaller is set to IgniteConfiguration.
In a case when you need to override default type and field IDs calculation or to plug in `BinarySerializer`, a `BinaryConfiguration` object should be set to `IgniteConfiguration`. This object allows to specify a global ID mapper and a global binary serializer as well as specify per-type mappers and serializers. Wildcards are supported for per-type configuration, in this case provided configuration will be applied to all types matching type name template.
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <property name=\"binaryConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.BinaryConfiguration\">\n            <property name=\"idMapper\" ref=\"globalIdMapper\"/>\n\n            <property name=\"typeConfigurations\">\n                <list>\n                    <bean class=\"org.apache.ignite.binary.BinaryTypeConfiguration\">\n                        <property name=\"typeName\" value=\"org.apache.ignite.examples.*\"/>\n                        <property name=\"serializer\" ref=\"exampleSerializer\"/>\n                    </bean>\n                </list>\n            </property>\n        </bean>\n    </property>\n...",
      "language": "xml",
      "name": "Configuring Binary Types"
    }
  ]
}
[/block]