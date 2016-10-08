+ Types
  + [`assemble`](#assemblepath-opts-cb)
  + [`combine`](#combinetypes-opts)
  + [`infer`](#inferval-opts)
  + [`parse`](#parseschema-opts)
  + [`Type`](#class-type)
  + Built-in types:
    + [`types.ArrayType`](#class-arraytypeattrs-opts)
    + `types.BooleanType`
    + `types.BytesType`
    + `types.DoubleType`
    + [`types.EnumType`](#class-enumtypeattrs-opts)
    + [`types.FixedType`](#class-fixedtypeattrs-opts)
    + `types.FloatType`
    + `types.IntType`
    + [`types.LogicalType`](#class-logicaltypeattrs-opts-types)
    + [`types.LongType`](#class-longtypeattrs-opts)
    + [`types.MapType`](#class-maptypeattrs-opts)
    + `types.NullType`
    + [`types.RecordType`](#class-recordtypeattrs-opts)
    + `types.StringType`
    + [`types.UnwrappedUnionType`](#class-unwrappeduniontypeattrs-opts)
    + [`types.WrappedUnionType`](#class-wrappeduniontypeattrs-opts)
+ Files and streams
  + [`createFileDecoder`](#createfiledecoderpath-opts)
  + [`createFileEncoder`](#createfileencoderpath-schema-opts)
  + [`extractFileHeader`](#extractfileheaderpath-opts)
  + Streams:
    + [`streams.BlockDecoder`](#class-blockdecoderopts)
    + [`streams.BlockEncoder`](#class-blockencoderschema-opts)
    + [`streams.RawDecoder`](#class-rawdecoderschema-opts)
    + [`streams.RawEncoder`](#class-rawencoderschema-opts)
+ IPC & RPC
  + [`Protocol`](#class-protocol)
  + [`Protocol.MessageEmitter`](#class-messageemitter)
  + [`Protocol.MessageListener`](#class-messagelistener)


## Types

### `assemble(path, [opts,] cb)`

+ `path` {String} Path to Avro IDL file.
+ `opts` {Object} Options:
  + `importHook(path, kind, cb)` {Function} Function called to load each file.
    The default will look up the files in the local file-system and load them
    via `fs.readFile`. `kind` is one of `'idl'`, `'protocol'`, or `'schema'`
    depending on the kind of import requested. *In the browser, no default
    is provided.*
  + `oneWayVoid` {Boolean} By default, using `void` as message response type is
    equivalent to passing `null`. When this option is set, messages with `void`
    response type will also be defined as one-way.
  + `reassignJavadoc` {Boolean} By default Javadoc comments become a `doc`
    attribute on the type, field, or message definition that follows. Setting
    this option will reassign a field type's Javadoc to its field, and a
    response type's Javadoc to its message.
+ `cb(err, attrs)` {Function} Callback. If an error occurred, its `path`
  property will contain the path to the file which caused it.

Assemble an IDL file into its attributes. These can then be passed to
[`parse`](#parseschema-opts) to create the corresponding protocol.

### `parse(schema, [opts])`

+ `schema` {Object|String} An Avro protocol or type schema, represented by one
  of:
  + A decoded schema object (e.g. `{type: 'array', items: 'int'}`).
  + A string containing a JSON-stringified schema (e.g. `'["null", "int"]'`).
  + A path to a file containing a JSON-stringified schema (e.g.
    `'./Schema.avsc'`). *This last option is not supported in the browser.*
+ `opts` {Object} Parsing options. The following keys are currently supported:
  + `assertLogicalTypes` {Boolean} The Avro specification mandates that we fall
    through to the underlying type if a logical type is invalid. When set, this
    option will override this behavior and throw an error when a logical type
    can't be applied.
  + `logicalTypes` {Object} Optional dictionary of
    [`LogicalType`](#class-logicaltypeattrs-opts-types). This can be used to
    support serialization and deserialization of arbitrary native objects.
  + `namespace` {String} Optional parent namespace.
  + `noAnonymousTypes` {Boolean} Throw an error if a named type (`enum`,
    `fixed`, `record`, or `error`) is missing its `name` field. By default
    anonymous types are supported; they behave exactly like their named
    equivalent except that they cannot be referenced and can be resolved by any
    compatible type.
  + `registry` {Object} Registry of predefined type names. This can for example
    be used to override the types used for primitives or to split a schema
    declaration over multiple files.
  + `typeHook(attrs, opts)` {Function} Function called before each type
    declaration or reference is parsed. The relevant decoded schema is
    available as first argument and the parsing options as second. This
    function can optionally return a type which will then be used in place of
    the result of parsing `schema`. Using this option, it is possible to
    customize the parsing process by intercepting the creation of any type.
    Here are a few examples of what is possible using a custom hook:
    + [Representing `enum`s as integers rather than strings.](https://gist.github.com/mtth/c0088c745de048c4e466#file-long-enum-js)
    + [Obfuscating all names inside a schema.](https://gist.github.com/mtth/c0088c745de048c4e466#file-obfuscate-js)
    + [Inlining fields to implement basic inheritance between records.](https://gist.github.com/mtth/c0088c745de048c4e466#file-inline-js)
  + `wrapUnions` {Boolean} Represent unions using a
    [`WrappedUnionType`](#class-wrappeduniontypeattrs-opts) instead of the
    default [`UnwrappedUnionType`](#class-unwrappeduniontypeattrs-opts).

Parse a schema and return an instance of the corresponding
[`Type`](#class-type) or [`Protocol`](#class-protocol).

### `combine(types, [opts])`

+ `types` {Array} Array of types to combine.
+ `opts` {Object} All the options of [`parse`](#parseschema-opts) are
  available, as well as:
  + `strictDefaults` {Boolean} When combining records with missing fields, the
    default behavior is to make such fields optional (wrapping their type
    inside a nullable union and setting their default to `null`). Activating
    this flag will instead combine the records into a map.

Merge multiple types into one. The resulting type will support all the input
types' values.

### `infer(val, [opts])`

+ `val` {...} The value used to infer the type.
+ `opts` {Object} Options. All the options of [`combine`](#combinetypes-opts)
  (and therefore also of [`parse`](#parseschema-opts)) are supported, along
  with:
  + `valueHook(val, opts)` Function called each time a type needs to be
    inferred from a value. This function should either return an alternate type
    to use, or `undefined` to proceed with the default inference logic.

Generate a type from a value.


### Class `Type`

"Abstract" base Avro type class; all implementations inherit from it.

##### `type.decode(buf, [pos,] [resolver])`

+ `buf` {Buffer} Buffer to read from.
+ `pos` {Number} Offset to start reading from.
+ `resolver` {Resolver} Optional resolver to decode values serialized from
  another schema. See [`createResolver`](#typecreateresolverwritertype) for how
  to create one.

Returns `{value: value, offset: offset}` if `buf` contains a valid encoding of
`type` (`value` being the decoded value, and `offset` the new offset in the
buffer). Returns `{value: undefined, offset: -1}` when the buffer is too short.

##### `type.encode(val, buf, [pos])`

+ `val` {...} The value to encode. An error will be raised if this isn't a
  valid `type` value.
+ `buf` {Buffer} Buffer to write to.
+ `pos` {Number} Offset to start writing at.

Encode a value into an existing buffer. If enough space was available in `buf`,
returns the new (non-negative) offset, otherwise returns `-N` where `N` is the
(positive) number of bytes by which the buffer was short.

##### `type.fromBuffer(buf, [resolver,] [noCheck])`

+ `buf` {Buffer} Bytes containing a serialized value of `type`.
+ `resolver` {Resolver} To decode values serialized from another schema. See
  [`createResolver`](#typecreateresolverwritertype) for how to create an
  resolver.
+ `noCheck` {Boolean} Do not check that the entire buffer has been read. This
  can be useful when using an resolver which only decodes fields at the start of
  the buffer, allowing decoding to bail early and yield significant performance
  speedups.

Deserialize a buffer into its corresponding value.

##### `type.toBuffer(val)`

+ `val` {...} The value to encode. It must be a valid `type` value.

Returns a `Buffer` containing the Avro serialization of `val`.

##### `type.fromString(str)`

+ `str` {String} String representing a JSON-serialized object.

Deserialize a JSON-encoded object of `type`.

##### `type.toString([val])`

+ `val` {...} The value to serialize. If not specified, this method will return
  a human-friendly description of `type`.

Serialize an object into a JSON-encoded string.

##### `type.isValid(val, [opts])`

+ `val` {...} The value to validate.
+ `opts` {Object} Options:
  + `errorHook(path, any, type)` {Function} Function called when an invalid
    value is encountered. When an invalid value causes its parent values to
    also be invalid, the latter do not trigger a callback. `path` will be an
    array of strings identifying where the mismatch occurred. This option is
    especially useful when dealing with complex records, for example to:
      + [Collect all paths to invalid nested values.](https://gist.github.com/mtth/fe006b5b001beeaed95f#file-collect-js)
      + [Throw an error with the full path to an invalid nested value.](https://gist.github.com/mtth/fe006b5b001beeaed95f#file-assert-js)
  + `noUndeclaredFields` {Boolean} When set, records with attributes that don't
    correspond to a declared field will be considered invalid. The default is
    to ignore any extra attributes.

Check whether `val` is a valid `type` value.

##### `type.clone(val, [opts])`

+ `val` {...} The object to copy.
+ `opts` {Object} Options:
  + `coerceBuffers` {Boolean} Allow coercion of JSON buffer representations
    into actual `Buffer` objects. When used with unwrapped unions, ambiguities
    caused by this coercion are always resolved in favor of the buffer type.
  + `fieldHook(field, any, type)` {Function} Function called when each record
    field is populated. The value returned by this function will be used
    instead of `any`. `field` is the current `Field` instance and `type` the
    parent type.
  + `qualifyNames` {Boolean} The branch's key in the union object should be the
    qualified name of its type, however some serializers incorrectly omit the
    namespace (which can cause collisions). Passing in this option will attempt
    to lookup unqualified names as well and return correctly qualified names.
    This option has no effect when used with unwrapped unions.
  + `wrapUnions` {Boolean} Allow wrapping of union values into their first
    matching branch. This option has no effect when used with unwrapped unions.

Deep copy a value of `type`.

##### `type.compare(val1, val2)`

+ `val1` {...} Value of `type`.
+ `val2` {...} Value of `type`.

Returns `0` if both values are equal according to their [sort
order][sort-order], `-1` if the first is smaller than the second , and `1`
otherwise. Comparing invalid values is undefined behavior.

##### `type.compareBuffers(buf1, buf2)`

+ `buf1` {Buffer} `type` value bytes.
+ `buf2` {Buffer} `type` value bytes.

Similar to [`compare`](#typecompareval1-val2), but doesn't require decoding
values.

##### `type.createResolver(writerType)`

+ `writerType` {Type} Writer type.

Create a resolver that can be be passed to the `type`'s
[`decode`](#typedecodebuf-pos-resolver) and
[`fromBuffer`](#typefrombufferbuf-resolver-nocheck) methods. This will enable
decoding values which had been serialized using `writerType`, according to the
Avro [resolution rules][schema-resolution]. If the schemas are incompatible,
this method will throw an error.

For example, assume we have the following two versions of a type:

```javascript
// A schema's first version.
const v1 = avro.parse({
  name: 'Person',
  type: 'record',
  fields: [
    {name: 'name', type: 'string'},
    {name: 'age', type: 'int'}
  ]
});

// The updated version.
const v2 = avro.parse({
  type: 'record',
  name: 'Person',
  fields: [
    {
      name: 'name', type: [
        'string',
        {
          name: 'Name',
          type: 'record',
          fields: [
            {name: 'first', type: 'string'},
            {name: 'last', type: 'string'}
          ]
        }
      ]
    },
    {name: 'phone', type: ['null', 'string'], default: null}
  ]
});
```

The two types are compatible since the `name` field is present in both (the
`string` can be promoted to the new `union`) and the new `phone` field has a
default value.

```javascript
//  We can therefore create a resolver.
const resolver = v2.createResolver(v1);

// And pass it whenever we want to decode from the old type to the new.
const buf = v1.toBuffer({name: 'Ann', age: 25});
const obj = v2.fromBuffer(buf, resolver); // === {name: {string: 'Ann'}, phone: null}
```

See the [advanced usage page](Advanced-usage) for more details on how schema
evolution can be used to significantly speed up decoding.

##### `type.random()`

Returns a random value of `type`.

##### `type.getName([asBranch])`

+ `asBranch` {Boolean} If `type` doesn't have a name, return its "type name"
  instead of `undefined`. (This method then returns the type's branch name when
  included in a union.)

Returns `type`'s fully qualified name if it exists, `undefined` otherwise.

##### `type.getTypeName()`

Returns `type`'s "type name" (e.g. `'int'`, `'record'`, `'fixed'`).

##### `type.getSchema([opts])`

+ `opts` {Object} Options:
  + `exportAttrs` {Boolean} Include aliases, field defaults, order, and logical
    type attributes in the returned schema.
  + `noDeref` {Boolean} Do not dereference any type names.

Returns `type`'s [canonical schema][canonical-schema] (as a string). This can
be used to compare schemas for equality.

##### `type.getFingerprint([algorithm])`

+ `algorithm` {String} Algorithm used to compute the hash. Defaults to `'md5'`.
  *Only `'md5'` is supported in the browser.*

Return a buffer identifying `type`.

##### `type.equals(other)`

+ `other` {...} Any object.

Check whether two types are equal (i.e. have the same canonical schema).

##### `Type.isType(any, [prefix,] ...)`

+ `any` {...} Any object.
+ `prefix` {String} If specified, this function will only return `true` if
  the type's type name starts with at least one of these prefixes. For example,
  `Type.isType(type, 'union', 'int')` will return `true` if and only if `type`
  is either a union type or integer type.

Check whether `any` is an instance of `Type`. This is similar to `any
instanceof Type` but will work across contexts (e.g. `iframe`s).

##### `Type.__reset(size)`

+ `size` {Number} New buffer size in bytes.

This method resizes the internal buffer used to encode all types. You can call
this method if you are encoding very large values and need to reclaim memory.
In some cases, it can also be beneficial to call this method at startup with a
sufficiently large buffer size to allow the JavaScript engine to better
optimize encoding.


#### Class `ArrayType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

##### `type.getItemsType()`

The type of the array's items.


#### Class `EnumType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.

##### `type.getSymbols()`

Returns a copy of the type's symbols (an array of strings representing the
`enum`'s valid values).


#### Class `FixedType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.

##### `type.getSize()`

The size in bytes of instances of this type.


#### Class `LogicalType(attrs, [opts,] [Types])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.
+ `Types` {Array} Optional of type classes. If specified, only these will be
  accepted as underlying type.

"Abstract class" used to implement custom types. To implement a new logical
type, the steps are:

+ Call `LogicalType`'s constructor inside your own subclass' to make sure the
  underlying type is property set up. Throwing an error anywhere inside your
  constructor will prevent the logical type from being used (the underlying
  type will be used instead).
+ Extend `LogicalType` in your own subclass (typically using `util.inherits`).
+ Override the following methods (prefixed with an underscore because they are
  internal to the class that defines them and should only be called by the
  internal `LogicalType` methods):
  + `_fromValue`
  + `_toValue`
  + `_resolve` (optional)
  + `_export` (optional)

See [here][logical-types] for more information. A couple sample implementations
are available as well:

+ [`DateType`](https://gist.github.com/mtth/1aec40375fbcb077aee7#file-date-js)
+ [`DecimalType`](https://gist.github.com/mtth/1aec40375fbcb077aee7#file-decimal-js)

##### `type.getUnderlyingType()`

Use this method to get the underlying Avro type. This can be useful when a
logical type can support different underlying types.

##### `type._fromValue(val)`

+ `val` {...} A value deserialized by the underlying type.

This method should return the converted value. *This method is abstract and
should be implemented but not called directly.*

##### `type._toValue(any)`

+ `any` {...} A derived value.

This method should return a value which can be serialized by the underlying
type. If `any` isn't a valid value for this logical type, you can either return
`undefined` or throw an exception (slower). *This method is abstract and should
be implemented but not called directly.*

##### `type._resolve(type)`

+ `type` {Type} The writer's type.

This method should return:

+ `undefined` if the writer's values cannot be converted.
+ Otherwise, a function which converts a value deserialized by the writer's
  type into a wrapped value for the current type.

*This method is abstract and should be implemented but not called directly.*

##### `type._export(attrs)`

+ `attrs` {Object} The type's raw exported attributes, containing `type` and
  `logicalType` keys.

This method should add attributes to be exported to the `attrs` object. These
will then be included into any [`type.getSchema`](#typegetschema-opts) calls
with `exportAttrs` set. *A default implementation exporting nothing is
provided.*


#### Class `LongType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

##### `LongType.__with(methods, [noUnpack])`

+ `methods` {Object} Method implementations dictionary keyed by method name,
  see below for details on each of the functions to implement.
+ `noUnpack` {Boolean} Do not automatically unpack bytes before passing them to
  the above `methods`' `fromBuffer` function and pack bytes returned by its
  `toBuffer` function.

This function provides a way to support arbitrary long representations. Doing
so requires implementing the following methods (a few examples are available
[here][custom-long]):

+ `fromBuffer(buf)`

  + `buf` {Buffer} Encoded long. If `noUnpack` is off (the default), `buf` will
    be an 8-byte buffer containing the long's unpacked representation.
    Otherwise, `buf` will contain a variable length buffer with the long's
    packed representation.

  This method should return the corresponding decoded long.

+ `toBuffer(val)`

  + `val` {...} Decoded long.

  If `noUnpack` is off (the default), this method should return an 8-byte
  buffer with the `long`'s unpacked representation. Otherwise, `toBuffer`
  should return an already packed buffer (of variable length).

+ `fromJSON(any)`

  + `any` {Number|...} Parsed value. To ensure that the `fromString` method
    works correctly on data JSON-serialized according to the Avro spec, this
    method should at least support numbers as input.

  This method should return the corresponding decoded long.

  It might also be useful to support other kinds of input (typically the output
  of the long implementation's `toJSON` method) to enable serializing large
  numbers without loss of precision (at the cost of violating the Avro spec).

+ `toJSON(val)`

  + `val` {...} Decoded long.

  This method should return the `long`'s JSON representation.

+ `isValid(val, [opts])`

  See [`Type.isValid`](#typeisvalidval-opts).

+ `compare(val1, val2)`

  See [`Type.compare`](#typecompareval1-val2).


#### Class `MapType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

##### `type.getValuesType()`

The type of the map's values (keys are always strings).


#### Class `RecordType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.

##### `type.getField(name)`

+ `name` {String} Field name.

Convenience method to retrieve a field by name. A field is an object with the
following methods:

+ `getAliases()`
+ `getDefault()`
+ `getName()`
+ `getOrder()`
+ `getType()`

##### `type.getFields()`

Returns a copy of the array of fields contained in this record.

##### `type.getRecordConstructor()`

The [`Record`](#class-record) constructor for instances of this type. Indeed,
each [`RecordType`](#class-recordtypeattrs-opts) generates a corresponding
`Record` constructor when its schema is parsed. This helps make decoding and
encoding records more efficient. This also lets us provide helpful methods on
decoded values (see below).

##### Class `Record(...)`

Calling the constructor directly can sometimes be a convenient shortcut to
instantiate new records of a given type. In particular, it will correctly
initialize all the missing record's fields with their default values.

###### `Record.getType()`

Convenience class method to get the record's type.

The `Record` prototype also exposes the following methods (available on each
decoded `record` value):

+ `record.clone([opts])`
+ `record.compare(val)`
+ `record.isValid([opts])`
+ `record.toBuffer()`
+ `record.toString()`


#### Class `UnwrappedUnionType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

This class is the default used to represent unions. Its values are decoded
without a wrapping object: `null` and `48` would be valid values for the schema
`["null", "int"]` (as opposed to `null` and `{'int': 48}` for wrapped unions).

This representation is usually more convenient and natural, however it isn't
able to guarantee correctness for all unions. For example, we wouldn't be able
to tell which branch the value `23` comes from in a schema `["int", "float"]`.
More concretely, a union can be represented using this class if it has at most
a single branch inside each of the categories below:

+ `'null'`
+ `'boolean'`
+ `'int'`, `'long'`, `'float'`, `'double'`
+ `'string'`, `'enum'`
+ `'bytes'`, `'fixed'`
+ `'array'`
+ `'map'`, `'record'`

So `['null', 'int']` and `['null', 'string', {type: 'array', items: 'string'}]`
are supported, but `['int', 'float']` and `['bytes', {name: 'Id', type:
'fixed', size: 2}]` are not.

Finally, note that by using logical types, it is possible to work around the
above requirements (by delegating the branch inference to the logical types
themselves).

##### `type.getTypes()`

The possible types that this union can take.


#### Class `WrappedUnionType(attrs, [opts])`

+ `attrs` {Object} Decoded type attributes.
+ `opts` {Object} Parsing options.

This class is the representation using for unions for types generated with
`parse`'s `wrapUnions` option set. It uses Avro's JSON encoding and is able to
correctly represent all unions: branch type information is never lost since it
is included in the decoded value.

##### `type.getTypes()`

The possible types that this union can take.

Additionally, each value decoded from a wrapped union exposes its corresponding
type via its constructor. This is also typically faster than calling
`Object.keys()` on the value when the active branch is unknown.

```javascript
const type = new avro.types.WrappedUnionType(['int', 'long']);
const val = type.fromBuffer(new Buffer([2, 8])); // == {long: 4}
const branchType = val.constructor.getBranchType() // == <LongType>
```


## Files and streams

The following convenience functions are available for common operations on
container files:

#### `extractFileHeader(path, [opts])`

+ `path` {String} Path to Avro container file.
+ `opts` {Object} Options:
  + `decode` {Boolean} Decode schema and codec metadata (otherwise they will be
    returned as bytes). Defaults to `true`.

Extract header from an Avro container file synchronously. If no header is
present (i.e. the path doesn't point to a valid Avro container file), `null` is
returned. *Not available in the browser.*

#### `createFileDecoder(path, [opts])`

+ `path` {String} Path to Avro container file.
+ `opts` {Object} Decoding options, passed to
  [`BlockDecoder`](#class-blockdecoderopts).

Returns a readable stream of decoded objects from an Avro container file. *Not
available in the browser.*

#### `createFileEncoder(path, schema, [opts])`

+ `path` {String} Destination path.
+ `schema` {Object|String|Type} Type used to serialize.
+ `opts` {Object} Encoding options, passed to
  [`BlockEncoder`](#class-blockencoderschema-opts).

Returns a writable stream of objects. These will end up serialized into an Avro
container file. *Not available in the browser.*

#### `createBlobDecoder(blob, [opts])`

+ `blob` {Blob} Binary blob.
+ `opts` {Object} Decoding options, passed to
  [`BlockDecoder`](#class-blockdecoderopts).

Returns a readable stream of decoded objects from an Avro container blob. *Only
available in the browser when using the full distribution.*

#### `createBlobEncoder(schema, [opts])`

+ `schema` {Object|String|Type} Type used to serialize.
+ `opts` {Object} Encoding options, passed to
  [`BlockEncoder`](#class-blockencoderschema-opts).

Returns a duplex stream of objects. Written values will end up serialized into
an Avro container blob which will be output as the stream's only readable
value. *Only available in the browser when using the full distribution.*


#### Class `BlockDecoder([opts])`

+ `opts` {Object} Decoding options. Available keys:
  + `codecs` {Object} Dictionary of decompression functions, keyed by codec
    name. A decompression function has the signature `fn(compressedData, cb)` where
    `compressedData` is a buffer of compressed data, and must call `cb(err,
    uncompressedData)` on completion. The default contains handlers for the
    `'null'` and `'deflate'` codecs.
  + `noDecode` {Boolean} Do not decode records before returning them.
  + `parseHook(attrs)` {Function} Function called to generate the type from the
    schema contained in the file. This can be used to pass in addtional options
    when parsing the schema (e.g. logical type information). See below for an
    example.

A duplex stream which decodes bytes coming from on Avro object container file.

Sample use of the `codecs` option to decode a Snappy encoded file using
[snappy](https://www.npmjs.com/package/snappy) (note [checksum
handling](https://avro.apache.org/docs/1.8.0/spec.html#snappy)):

```javascript
const snappy = require('snappy');

const blockDecoder = new avro.streams.BlockDecoder({
  codecs: {
    snappy: (buf, cb) => {
      // The checksum is ignored here, we could use it for validation instead.
      return snappy.uncompress(buf.slice(0, buf.length - 4), cb);
    }
  }
});
```

Note that the `BlockDecoder`'s `opts` aren't used when parsing the writer's
type. A `parseHook` should be used instead. The example below shows how to
instantiate a type with the `wrapUnions` option set:

```javascript
const decoder = new avro.streams.BlockDecoder({
  parseHook: function (attrs) { return avro.parse(attrs, {wrapUnions: true}); }
});
```


##### Event `'metadata'`

+ `type` {Type} The type used to write the file.
+ `codec` {String} The codec's name.
+ `header` {Object} The file's header, containing in particular the raw schema
  and codec.

This event is guaranteed to be emitted before the first `'data'` event.

##### Event `'data'`

+ `data` {...} Decoded element or raw bytes.

##### `BlockDecoder.getDefaultCodecs()`

Get built-in decompression functions (currently `null` and `deflate`).


#### Class `BlockEncoder(schema, [opts])`

+ `schema` {Object|String|Type} Schema used for encoding. Argument parsing
  logic is the same as for [`parse`](#parseschema-opts).
+ `opts` {Object} Encoding options. Available keys:
  + `blockSize` {Number} Maximum uncompressed size of each block data. A new
    block will be started when this number is exceeded. If it is too small to
    fit a single element, it will be increased appropriately. Defaults to 64kB.
  + `codec` {String} Name of codec to use for encoding. See `codecs` option
    below to support arbitrary compression functions.
  + `codecs` {Object} Dictionary of compression functions, keyed by codec
    name. A compression function has the signature `fn(uncompressedData, cb)` where
    `uncompressedData` is a buffer of uncompressed data, and must call `cb(err,
    compressedData)` on completion. The default contains handlers for the
    `'null'` and `'deflate'` codecs.
  + `omitHeader` {Boolean} Don't emit the header. This can be useful when
    appending to an existing container file. Defaults to `false`.
  + `syncMarker` {Buffer} 16 byte buffer to use as synchronization marker
    inside the file. If unspecified, a random value will be generated.

A duplex stream to create Avro container object files.

##### Event `'data'`

+ `data` {Buffer} Serialized bytes.

##### `BlockEncoder.getDefaultCodecs()`

Get built-in compression functions (currently `null` and `deflate`).


#### Class `RawDecoder(schema, [opts])`

+ `schema` {Object|String|Type} Writer schema. Required since the input doesn't
  contain a header. Argument parsing logic is the same as for
  [`parse`](#parseschema-opts).
+ `opts` {Object} Decoding options. Available keys:
  + `decode` {Boolean} Whether to decode records before returning them.
    Defaults to `true`.

A duplex stream which can be used to decode a stream of serialized Avro objects
with no headers or blocks.

##### Event `'data'`

+ `data` {...} Decoded element or raw bytes.


#### Class `RawEncoder(schema, [opts])`

+ `schema` {Object|String|Type} Schema used for encoding. Argument parsing
  logic is the same as for [`parse`](#parseschema-opts).
+ `opts` {Object} Encoding options. Available keys:
  + `batchSize` {Number} To increase performance, records are serialized in
    batches. Use this option to control how often batches are emitted. If it is
    too small to fit a single record, it will be increased automatically.
    Defaults to 64kB.

The encoding equivalent of `RawDecoder`.

##### Event `'data'`

+ `data` {Buffer} Serialized bytes.


# IPC & RPC

Avro also defines a way of executing remote procedure calls. We expose this via
an API modeled after node.js' core [`EventEmitter`][event-emitter].

#### Class `Protocol`

`Protocol` instances are obtained by [`parse`](#parseschema-opts)-ing a
[protocol declaration][protocol-declaration] and provide a way of sending
remote messages (for example to another machine, or another process on the same
machine). For this reason, instances of this class are very similar to
`EventEmitter`s, exposing both [`emit`](#protocolemitname-req-emitter) and
[`on`](#protocolonname-handler) methods.

Being able to send remote messages (and to do so efficiently) introduces a few
differences however:

+ The types used in each event (for both the emitted message and its response)
  must be defined upfront in the protocol's declaration.
+ The arguments emitted with each event must match the ones defined in the
  protocol. Similarly, handlers are guaranteed to be called only with these
  matching arguments.
+ Events are one-to-one: they have exactly one response (unless they are
  declared as one-way, in which case they have none).

##### `protocol.emit(name, req, emitter, cb)`

+ `name` {String} Name of the message to emit. If this message is sent to a
  `Protocol` instance with no handler defined for this name, an "unsupported
  message" error will be returned.
+ `req` {Object} Request value, must correspond to the message's declared
  request type.
+ `emitter` {MessageEmitter} Emitter used to send the message. See
  [`createEmitter`](#protocolcreateemittertransport-opts) for how to obtain
  one.
+ `cb(err, res)` {Function} Function called with the remote call's response
  (and eventual error) when available. If not specified and an error occurs,
  the error will be emitted on `emitter` instead.

Send a message. This is always done asynchronously. This method is a simpler
version of [`emitter.emitMessage`](#emitteremitmessagename-envelope-opts-cb),
providing convenience functionality such as converting string errors to `Error`
objects.

##### `protocol.on(name, handler)`

+ `name` {String} Message name to add the handler for. An error will be thrown
  if this name isn't defined in the protocol. At most one handler can exist for
  a given name (any previously defined handler will be overwritten).
+ `handler(req, listener, cb)` {Function} Handler, called each time a message
  with matching name is received. The `listener` argument will be the
  corresponding `MessageListener` instance. The final callback argument
  `cb(err, res)` should be called to send the response back to the emitter.

Add a handler for a given message.

##### `protocol.createEmitter(transport, [opts])`

+ `transport` {Duplex|Object|Function} The transport used to communicate with
  the remote listener. Multiple argument types are supported, see below.
+ `opts` {Object} Options.
  + `cache` {Object} Cache of remote server protocols. This can be used in
    combination with existing emitters' [`emitter.getCache`](#emittergetcache)
    to avoid performing too many handshakes.
  + `endWritable` {Boolean} Set this to `false` to prevent the transport's
    writable stream from being `end`ed when the emitter is destroyed (for
    stateful transports) or when a request is sent (for stateless transports).
    Defaults to `true`.
  + `noPing` {Boolean} Do not emit a ping request when the emitter is created.
    For stateful transports this will assume that a connection has already been
    established, for stateless transports this will delay handshakes until the
    first message is sent.
  + `objectMode` {Boolean} Expect a transport in object mode. Instead of
    exchanging buffers, objects `{id, payload}` will be written and expected.
    This can be used to implement custom transport encodings.
  + `scope` {String} Scope used to multiplex messages accross a shared
    connection. There should be at most one emitter or listener per scope on a
    single stateful transport. Matching emitter/listener pairs should have
    matching scopes. Scoping isn't supported on stateless transports.
  + `serverFingerprint` {Buffer} Fingerprint of remote server to use for the
    initial handshake. This will only be used if the corresponding adapter
    exists in the cache.
  + `strictErrors` {Boolean} Disable conversion of string errors to `Error`
    objects.
  + `timeout` {Number} Default timeout in milliseconds used when sending
    requests. It is possible to override this per request via the
    [`emitter.emitMessage`](#emitteremitmessagename-envelope-opts-cb) function.
    Specify `0` for no timeout. Defaults to `10000`.

Generate a [`MessageEmitter`](#class-messageemitter) for this protocol. This
emitter can then be used to communicate with a remote server of compatible
protocol.

There are two major types of transports:

+ Stateful: a pair of binary streams `{readable, writable}`. As a convenience
  passing a single duplex stream is also supported and equivalent to passing
  `{readable: duplex, writable: duplex}`.

+ Stateless: stream factory `fn(cb)` which should return a writable stream and
  call its callback argument with a readable stream (when available).

##### `protocol.createListener(transport, [opts])`

+ `transport` {Duplex|Object|Function} Similar to
  [`createEmitter`](#protocolcreateemittertransport-opts-cb)'s corresponding
  argument, except that readable and writable roles are reversed for stateless
  transports.
+ `opts` {Object} Options.
  + `cache` {Object} Cache of remote client protocols. This can be used in
    combination with existing listeners'
    [`listener.getCache`](#listenergetcache) to avoid performing too many
    handshakes.
  + `endWritable` {Boolean} Set this to `false` to prevent the transport's
    writable stream from being `end`ed when the emitter is destroyed (for
    stateful transports) or when a response is sent (for stateless transports).
    Defaults to `true`.
  + `objectMode` {Boolean} Expect a transport in object mode. Instead of
    exchanging buffers, objects `{id, payload}` will be written and expected.
    This can be used to implement custom transport encodings.
  + `scope` {String} Scope used to multiplex messages accross a shared
    connection. There should be at most one emitter or listener per scope on a
    single stateful transport. Matching emitter/listener pairs should have
    matching scopes. Scoping isn't supported on stateless transports.
    + `strictErrors` {Boolean} Disable automatic conversion of `Error` objects to
    strings. When set, the returned error parameter must either be a valid
    union branch or `undefined`.

Generate a [`MessageListener`](#class-messagelistener) for this protocol. This
listener can be used to respond to messages emitted from compatible protocols.

##### `protocol.getHandler(name)`

+ `name` {String} Message name.

Get the function called each time a message is received for this protocol, or
`undefined` if no handler was set for this message.

##### `protocol.subprotocol()`

Returns a copy of the original protocol, which inherits all its handlers.
Adding new handlers to a subprotocol won't affect the parent protocol.

##### `protocol.getMessage(name)`

+ `name` {String} Message name.

Get a single message from this protocol.

##### `protocol.getMessages()`

Retrieve all the messages defined in the protocol. Each message is an object
with the following methods:

+ `getName()`
+ `getRequestType()`
+ `getResponseType()`
+ `getErrorType()`
+ `isOneWay()`

##### `protocol.getName()`

Returns the protocol's fully qualified name.

##### `protocol.getType(name)`

+ `name` {String} A type's fully qualified name.

Convenience function to retrieve a type defined inside this protocol. Returns
`undefined` if no type exists for the given name.

##### `protocol.getSchema([opts])`

+ `opts` {Object} Same options as [`Type.getSchema`](#).

Returns `protocol`'s canonical schema.

##### `protocol.getFingerprint([algorithm])`

+ `algorithm` {String} Algorithm used to generate the protocol's fingerprint.
  Defaults to `'md5'`. *Only `'md5'` is supported in the browser.*

Returns a buffer containing the protocol's [fingerprint][].

##### `protocol.equals(other)`

+ `other` {...} Any object.

Check whether the argument is equal to `protocol` (w.r.t canonical
representations).


#### Class `MessageEmitter`

Instance of this class are [`EventEmitter`s][event-emitter], with the following
events:

##### Event `'handshake'`

+ `request` {Object} Handshake request.
+ `response` {Object} Handshake response.

Emitted when the server's handshake response is received.

##### Event `'eot'`

End of transmission event, emitted after the client is destroyed and there are
no more pending requests.

##### `emitter.emitMessage(name, envelope, [opts,] cb)`

+ `name` {String} Message name.
+ `envelope` {Object} Message contents `{header, request}`.
+ `opts` {Object} Call options, currently the following are supported:
  + `timeout` {Number}
+ `cb(err, envelope, meta)` {Function} Callback.

Send a message. This function provides a lower level API than
[`protocol.emit`](#protocolemitname-req-emitter-cb); for example it exposes
message headers and the timeout parameter.

##### `emitter.getCache()`

Get the emitter's cache of remote protocols. This cache can be reused to
instantiate new emitters and avoid having to perform additional handshakes.

##### `emitter.getPending()`

Get the number of pending calls (i.e. the number of messages emittes which
haven't yet had a response).

##### `emitter.getProtocol()`

Get the emitter's underlying protoocl.

##### `emitter.getTimeout()`

Get the emitter's default timeout.

##### `emitter.isDestroyed()`

Check whether the listener was destroyed.

##### `emitter.destroy([noWait])`

+ `noWait` {Boolean} Cancel any pending requests. By default pending requests
  will still be honored.

Disable the emitter.


#### Class `MessageListener`

Listeners are the receiving-side equivalent of `MessageEmitter`s and are also
[`EventEmitter`s][event-emitter], with the following events:

##### Event `'handshake'`

+ `request` {Object} Handshake request.
+ `response` {Object} Handshake response.

Emitted right before the server sends a handshake response.

##### Event `'eot'`

End of transmission event, emitted after the listener is destroyed and there are
no more responses to send.

##### `listener.getCache()`

Get the listener's cache of remote protocols. This cache can be reused to
instantiate new listeners and avoid having to perform additional handshakes.

##### `listener.getPending()`

Get the number of pending calls (i.e. the number of handler calls which haven't
yet returned).

##### `listener.getProtocol()`

Get the listener's underlying protoocl.

##### `listener.isDestroyed()`

Check whether the listener was destroyed.

##### `listener.onMessage(fn)`

+ `fn(name, envelope, meta, cb)` {Function} Handler. The callback `cb` should
  be called as: `cb(err, envelope)` where `err` is an eventual error and
  `envelope` an object `{header, error, response}`.

Add a handler to be called each time a message arrives in this listener.

##### `listener.destroy([noWait])`

+ `noWait` {Boolean} Don't wait for all pending responses to have been sent.

Disable this listener and release underlying streams. In general you shouldn't
need to call this: stateless listeners will be destroyed automatically when a
response is sent, and stateful listeners are best destroyed from the client's
side.


[canonical-schema]: https://avro.apache.org/docs/current/spec.html#Parsing+Canonical+Form+for+Schemas
[schema-resolution]: https://avro.apache.org/docs/current/spec.html#Schema+Resolution
[sort-order]: https://avro.apache.org/docs/current/spec.html#order
[fingerprint]: https://avro.apache.org/docs/current/spec.html#Schema+Fingerprints
[custom-long]: Advanced-usage#custom-long-types
[logical-types]: Advanced-usage#logical-types
[framing-messages]: https://avro.apache.org/docs/current/spec.html#Message+Framing
[event-emitter]: https://nodejs.org/api/events.html#events_class_events_eventemitter
[protocol-declaration]: https://avro.apache.org/docs/current/spec.html#Protocol+Declaration
