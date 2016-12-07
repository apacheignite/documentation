* [Basic Concepts](#basic-concepts)
* [BinaryObject Equality](r#binaryobject-equality)
* [Configuring Binary Objects](#configuring-binary-objects)
* [BinaryObject Cache API](#binaryobject-cache-api)
* [Modifying Binary Objects Using BinaryObjectBuilder](#modifying-binary-objects-using-binaryobjectbuilder)
* [BinaryObject Type Metadata](#binaryobject-type-metadata)
* [BinaryObject and CacheStore](#binaryobject-and-cachestore)
* [Binary Name Mapper and Binary ID Mapper](#binary-name-mapper-and-binary-id-mapper)
[block:api-header]
{
  "type": "basic",
  "title": "Basic Concepts"
}
[/block]
Starting from v1.5 Ignite introduced a new concept of storing data in caches called `BinaryObjects`. There are several advantages that the new serialization format provides:
 * It enables you to read an arbitrary field from an object's serialized form without full object deserialization. This ability completely removes the requirement to have the cache key and value classes deployed on the server node's classpath. 
 * It enables you to add and remove fields from objects of the same type. Given that server nodes do not have model classes definitions, this ability allows dynamic change to an objects structure, and even allows multiple clients with different versions of class definitions to co-exist.
 * It enables you to construct new objects based on a type name without having class definitions at all, hence allowing dynamic type creation.

Binary objects can be used only when default binary marshaller is used (i.e. no other marshaller is set to the configuration explicitly).
[block:callout]
{
  "type": "info",
  "body": "There are several restrictions that are implied by the `BinaryObject` format implementation:\n * Internally Ignite does not write field and type names, but uses a lower-case name hash to identify a field or a type. This means that fields or types with the same name hash are not allowed. Even though serialization will not work out-of-the box in the case of hash collision, Ignite provides a way to resolve this collision at the configuration level.\n * For the same reason `BinaryObject` format does not allow same field names on different levels of a class hierarchy.\n * `Externalizable` interface is ignored by default. If `BinaryObject` format is used, `Externalizable` classes will be written the same way as if they were `Serializable`, without `writeExternal()` and `readExternal()` methods. If for some reason this does not work for you, you should implement `Binarylizable` interface for your classes, plug in a custom `BinarySerializer` or switch to the `OptimizedMarshaller`",
  "title": "Restrictions"
}
[/block]
The `IgniteBinary` facade, which can be obtained from an instance of Ignite, contains all necessary methods to work with binary objects.
[block:api-header]
{
  "type": "basic",
  "title": "BinaryObject Equality"
}
[/block]
When an object is translated to the binary format, Ignite captures it's hash code and stores it alongside with the binary object fields. This way a proper and consistent hash code can be provided on all nodes in a cluster for any object. For the `equals` comparison, Ignite by default relies on binary representation of the serialized object.
[block:callout]
{
  "type": "warning",
  "body": "Note that since _by default_ `equals` works by comparing serialized forms of objects, it:\n * Compares all the fields in an object\n * Depends on the order in which fields are serialized",
  "title": "Binary Equals"
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "BinaryObjectBuilder hash code harness"
}
[/block]
When a `BinaryObject` is created with `BinaryObjectBuilder` by specifying field values and type name or id, hash code for it is specified via builder's method `hashCode(int)` bearing a parameter.
[block:callout]
{
  "type": "warning",
  "title": "Hash code must be explicitly set with BinaryObjectBuilder",
  "body": "All binary objects must have their hash codes set in order to be used as keys. An attempt of any action with a key created with `BinaryObjectBuilder` and missing hash code will result into failure."
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Configuring Binary Objects"
}
[/block]
In the vast majority of use-cases there is no need to additionally configure binary objects. `BinaryObject` marshaller is enabled by default when no other marshaller is set to IgniteConfiguration.
In a case when you need to override default type and field IDs calculation or to plug in `BinarySerializer`, a `BinaryConfiguration` object should be set to `IgniteConfiguration`. This object allows to specify a global name mapper, a global ID mapper and a global binary serializer as well as specify per-type mappers and serializers. Wildcards are supported for per-type configuration, in this case provided configuration will be applied to all types matching type name template.
[block:code]
{
  "codes": [
    {
      "code": "<bean id=\"ignite.cfg\" class=\"org.apache.ignite.configuration.IgniteConfiguration\">\n    <property name=\"binaryConfiguration\">\n        <bean class=\"org.apache.ignite.configuration.BinaryConfiguration\">\n            <property name=\"nameMapper\" ref=\"globalNameMapper\"/>\n          \n            <property name=\"idMapper\" ref=\"globalIdMapper\"/>\n\n            <property name=\"typeConfigurations\">\n                <list>\n                    <bean class=\"org.apache.ignite.binary.BinaryTypeConfiguration\">\n                        <property name=\"typeName\" value=\"org.apache.ignite.examples.*\"/>\n                        <property name=\"serializer\" ref=\"exampleSerializer\"/>\n                    </bean>\n                </list>\n            </property>\n        </bean>\n    </property>\n...",
      "language": "java",
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
  "body": "Note that not all types will be represented as `BinaryObject` when `withKeepBinary()` flag is enabled. There is a set of 'platform' types that includes primitive types, String, UUID, Date, Timestamp, BigDecimal, Collections, Maps and arrays of thereof that will never be represented as a `BinaryObject`.\n\nNote that in the example below key type `Integer` does not change because it is a platform type.",
  "title": "Platform Types"
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
  "title": "Modifying Binary Objects Using BinaryObjectBuilder"
}
[/block]
`BinaryObject` instances are immutable. An instance of `BinaryObjectBuilder`must be used in order to update fields and create a new `BinaryObject`.

An instance of `BinaryObjectBuilder` can be obtained from `IgniteBinary` facade. The builder may be created using a type name, in this case the returned builder will contain no fields, or it may be created using an existing `BinaryObject`, in this case the returned builder will copy all the fields from the given `BinaryObject`.

Another way to get an instance of `BinaryObjectBuilder` is to call `toBuilder()` on an existing instance of a `BinaryObject`. This will also copy all data from the `BinaryObject` to the created builder.
[block:callout]
{
  "type": "danger",
  "body": "Note that it is important that a proper hash code be set to `BinaryObjectBuilder` if constructed `BinaryObject` will be used as a cache key, because builder does not calculate hash code automatically and returned `BinaryObject` will have zero hash code otherwise.",
  "title": "BinaryObjectBuilder and Hash Code"
}
[/block]
Below is an example of using `BinaryObject` API to process data on server nodes without having user classes deployed on servers and without actual data deserialization.
[block:code]
{
  "codes": [
    {
      "code": "cache.<Integer, BinaryObject>withKeepBinary().invoke(\n    new CacheEntryProcessor<Integer, BinaryObject, Void>() {\n        @Override Void process(\n            MutableEntry<Integer, BinaryObject> entry, Object... args) {\n            // Create builder from the old value.\n            BinaryObjectBuilder bldr = entry.getValue().toBuilder();\n            \n            //Update the field in the builder.\n            bldr.setField(\"name\", \"Ignite\");\n            \n            // Set new value to the entry.\n            entry.setValue(bldr.build());\n                \n            return null;\n        }\n    });",
      "language": "java",
      "name": "BinaryObject Inside EntryProcessor"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "BinaryObject Type Metadata"
}
[/block]
As it was mentioned above, binary object structure may be changed at runtime hence it may also be useful to get information about a particular type that is stored in a cache such as field names, field type names & affinity field name. Ignite facilitates this requirement via the `BinaryType` interface. 

This interface also introduces a faster version of field getter called `BinaryField`. The concept is analogous to java reflection and allows to cache certain information about the field being read in the `BinaryField` instance, which is useful when reading the same field from a large collection of binary objects.
[block:code]
{
  "codes": [
    {
      "code": "Collection<BinaryObject> persons = getPersons();\n\nBinaryField salary = null;\n\ndouble total = 0;\nint cnt = 0;\n\nfor (BinaryObject person : persons) {\n    if (salary == null)\n        salary = person.type().field(\"salary\");\n\n    total += salary.value(person);\n    cnt++;\n}\n\ndouble avg = total / cnt;",
      "language": "java",
      "name": "BinaryField Example"
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
Setting `withKeepBinary()` on the cache API does not affect the way user objects are passed to a `CacheStore`. This is intentional because in most cases a single `CacheStore` implementation works either with deserialized classes, or with `BinaryObject` representations. To control the way objects are passed to the store then the `storeKeepBinary` flag on `CacheConfiguration` should be used. When this flag is set to `false`, deserialized values will be passed to the store, otherwise `BinaryObject` representations will be used.

Below is an example pseudo-code implementation of a store working with `BinaryObject`:
[block:code]
{
  "codes": [
    {
      "code": "public class CacheExampleBinaryStore extends CacheStoreAdapter<Integer, BinaryObject> {\n    @IgniteInstanceResource\n    private Ignite ignite;\n\n    /** {@inheritDoc} */\n    @Override public BinaryObject load(Integer key) {\n        IgniteBinary binary = ignite.binary();\n\n        List<?> rs = loadRow(key);\n\n        BinaryObjectBuilder bldr = binary.builder(\"Person\");\n\n        for (int i = 0; i < rs.size(); i++)\n            bldr.setField(name(i), rs.get(i));\n\n        return bldr.build();\n    }\n\n    /** {@inheritDoc} */\n    @Override public void write(Cache.Entry<? extends Integer, ? extends BinaryObject> entry) {\n        BinaryObject obj = entry.getValue();\n\n        BinaryType type = obj.type();\n\n        Collection<String> fields = type.fieldNames();\n        \n        List<Object> row = new ArrayList<>(fields.size());\n\n        for (String fieldName : fields)\n            row.add(obj.field(fieldName));\n        \n        saveRow(entry.getKey(), row);\n    }\n}",
      "language": "java",
      "name": "Binary CacheStore Implementation"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Binary Name Mapper and Binary ID Mapper"
}
[/block]
Internally, Ignite never writes full strings for field or type names. Instead, for performance reasons, Ignite writes integer hash codes for type and field names. It has been tested that  hash code conflicts for the type names or the field names within the same type are virtually non-existent and, to gain performance, it is safe to work with hash codes. For the cases when hash codes for different types or fields actually do collide, `BinaryNameMapper` and `BinaryIdMapper` allow to override the automatically generated hash code IDs for the type and field names.

`BinaryNameMapper` - maps type/class and field names to different names.
`BinaryIdMapper` - maps given from `BinaryNameMapper` type and filed name to ID that will be used by Ignite in internals.

Ignite provides the following out-of-the-box mappers implementation:
* `BinaryBasicNameMapper` - a basic implementation of `BinaryNameMapper` that returns a full or a simple name of given class depending on used `setSimpleName(boolean useSimpleName)` property.
* `BinaryBasicIdMapper` - a basic implementation of `BinaryIdMapper`. It has `setLowerCase(boolean isLowerCase)` configuration property. If the property is set to `false` then a hash code of given type or field name will be returned. If the property is set to `true` then a hash code of given type or field name in lower case will be returned.

If you are using solely Java client and do not specify mappers in `BinaryConfiguration` then Ignite will use `BinaryBasicNameMapper` and `simpleName` property will be set to `false`, and `BinaryBasicIdMapper` and `lowerCase` property will be set to `true`.

If you are using .Net or C++ client and do not specify mappers in `BinaryConfiguration` then Ignite will use `BinaryBasicNameMapper` and `simpleName` property will be set to `true`, and `BinaryBasicIdMapper` and `lowerCase` property will be set to `true`. 

By default there is no need to configure anything if you uses solely Java, .NET or C++. Mappers need to be configured if there is a tricky name conversion when platform interoperability is needed.