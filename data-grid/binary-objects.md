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
The `IgniteBinary` facade, which can be obtained from an instance of Ignite, contains all necessary methods to work with binary objects.
[block:api-header]
{
  "type": "basic",
  "title": "BinaryObject Equality"
}
[/block]
When an object is translated to the binary format, Ignite captures it's hash code and stores it alongside with the binary object fields. This way a proper and consistent hash code can be provided on all nodes in a cluster for any object. For the `equals` comparison, Ignite relies on binary representation of the serialized object.
[block:callout]
{
  "type": "warning",
  "title": "Binary Equals",
  "body": "Note that since `equals` works by comparing serialized forms of objects, it:\n * Compares all the fields in an object\n * Depends on the order in which fields are serialized"
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

[block:api-header]
{
  "type": "basic",
  "title": "BinaryObject Cache API"
}
[/block]
By default Ignite works with deserialized values as it is the most common use-case. To enable `BinaryObject` processing, a user needs to obtain an instance of `IgniteCache` using `withKeepBinary()` method. When enabled, this flag will ensure that objects returned from the cache will be in `BinaryObject` format, when possible. The same will stand for values being passed to `EntryProcessor` and `CacheInterceptor`.
[block:callout]
{
  "type": "info",
  "title": "Platform Types",
  "body": "Note that not all types will be represented as `BinaryObject` when `withKeepBinary()` flag is enabled. There is a set of 'platform' types that includes primitive types, String, UUID, Date, Timestamp, BigDecimal, Collections, Maps and arrays of thereof that will never be represented as a `BinaryObject`."
}
[/block]

[block:code]
{
  "codes": [
    {
      "code": "// Create a regular Person object and put it to the cache.\nPerson person = buildPerson(personId);\nignite.cache(\"myCache\").put(personId, person);\n\n// Get an instance of binary-enabled cache.\nIgniteCache<Integer, BinaryObject> binaryCache = ignite.cache(\"myCache\").withKeepBinary();\n\n// Get the above person object in the BinaryObject format.\nBinaryObject binaryPerson = binaryCache.get(personId);",
      "language": "java",
      "name": "Getting BinaryObject"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Modifying binary objects using BinaryObjectBuilder"
}
[/block]
`BinaryObject` instances are immutable. An instance of `BinaryObjectBuilder`must be used in order to update fields and create a new `BinaryObject`.

An instance of `BinaryObjectBuilder` can be obtained from `IgniteBinary` facade. The builder may be created using a type name, in this case returned builder will contain no fields, or it may be created using an existing `BinaryObject`, in this case the returned builder will copy all the fields from the given `BinaryObject`.

Another way to get an instance of `BinaryObjectBuilder` is to call `toBuilder()` on an existing instance of a `BinaryObject`. This will also copy all data from the `BinaryObject` to the created builder.
[block:callout]
{
  "type": "danger",
  "title": "BinaryObjectBuilder And Hash Code",
  "body": "Note that it is important that a proper hash code be set to `PortableBuilder` if constructed `BinaryObject` will be used as a cache key, because builder does not calculate hash code automatically and returned `BinaryObject` will have zero hash code otherwise."
}
[/block]
Below is an example of using `BinaryObject` API to process data on server nodes without having user classes deployed on servers and without actual data deserialization.
[block:code]
{
  "codes": [
    {
      "code": "cache.withKeepBinary().invoke(\n    new CacheEntryProcessor<Integer, BinaryObject, Void>() {\n        @Override Void process(\n            MutableEntry<Integer, BinaryObject> entry, Object... args) {\n            // Create builder from the old value.\n            BinaryObjectBuilder bldr = entry.getValue().toBuilder();\n            \n            //Update the field in the builder.\n            bldr.setField(\"name\", \"Ignite\");\n            \n            // Set new value to the entry.\n            entry.setValue(bldr.build());\n                \n            return null;\n        }\n    });",
      "language": "text",
      "name": "BinaryObject Inside EntryProcessor"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "BinaryObject and CacheStore"
}
[/block]
Setting `withKeepBinary()` on the cache API does not affect the way user objects are passed to a `CacheStore`. This is done on purpose because in most cases a single `CacheStore` implementation works either with deserialized classes, or with `BinaryObject` representation. To control the way objects are passed to the store, the `storeKeepBinary` flag on `CacheConfiguration` should be used. When this flag is set to `false`, deserialized values will be passed to the store, otherwise `BinaryObject`s will be used.