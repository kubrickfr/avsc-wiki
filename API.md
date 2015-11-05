+ [Parsing schemas](#parsing-schemas)
+ [Avro types](#avro-types)
+ [Records](#records)
+ [Files and streams](#files-and-streams)


## Parsing schemas

### `parse(schema, [opts])`

+ `schema` {Object|String} An Avro schema, represented by one of:
  + A string containing a JSON-stringified schema (e.g. `'["null", "int"]'`).
  + A path to a file containing a JSON-stringified schema (e.g.
    `'./Schema.avsc'`).
  + A decoded schema object (e.g. `{type: 'array', items: 'int'}`).
+ `opts` {Object} Parsing options. The following keys are currently supported:
  + `namespace` {String} Optional parent namespace.
  + `registry` {Object} Optional registry of predefined type names. This can
    for example be used to override the types used for primitives.
  + `logicalTypes` {Object} Optional dictionary of
    [`LogicalType`](#class-logicaltypeattrs-opts-types). This can be used to
    support serialization and deserialization of arbitrary native objects. See
    [here][logical-types] for more information.
  + `typeHook(schema, opts)` {Function} Function called before each new type is
    instantiated. The relevant schema is available as first argument and the
    parsing options as second. This function can optionally return a type which
    will then be used in place of the result of parsing `schema`.

Parse a schema and return an instance of the corresponding
[`Type`](#class-type).


## Avro types

All the classes below are available in the `avsc.types` namespace:

+ [`Type`](#class-type)
+ Primitive types:
  + `BooleanType`
  + `BytesType`
  + `DoubleType`
  + `FloatType`
  + `IntType`
  + [`LongType`](#class-longtypeattrs-opts)
  + `NullType`
  + `StringType`
+ Complex types:
  + [`ArrayType`](#class-arraytypeattrs-opts)
  + [`EnumType`](#class-enumtypeattrs-opts)
  + [`FixedType`](#class-fixedtypeattrs-opts)
  + [`MapType`](#class-maptypeattrs-opts)
  + [`RecordType`](#class-recordtypeattrs-opts)
  + [`UnionType`](#class-uniontypeattrs-opts)
+ [`LogicalType`](#class-logicaltypeattrs-opts-types)


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

+ `val` {Object} The value to encode. It must be a valid `type` value.

Returns a `Buffer` containing the Avro serialization of `val`.

##### `type.fromString(str)`

+ `str` {String} String representing a JSON-serialized object.

Deserialize a JSON-encoded object of `type`.

##### `type.toString([val])`

+ `val` {...} The value to serialize. If not specified, this method will return
  the [canonical version][canonical-schema] of `type`'s schema instead (which
  can then be used to compare schemas for equality).

Serialize an object into a JSON-encoded string.

##### `type.isValid(val, [opts])`

+ `val` {...} The value to validate.
+ `opts` {Object} Options:
  + `errorHook(path, obj, type)` {Function} Function called when an invalid
    value is encountered. When an invalid value causes its parent values to
    also be invalid, the latter do not trigger a callback. `path` will be an
    array of strings identifying where the mismatch occurred. See below for a
    few examples.

Check whether `val` is a valid `type` value.

For complex schemas, it can be difficult to figure out which part(s) of `val`
are invalid. The `errorHook` option provides access to more information about
these mismatches. We illustrate a few use-cases below:

```javascript
// A sample schema.
var personType = avsc.parse({
  type: 'record',
  name: 'Person',
  fields: [
    {name: 'age', type: 'int'},
    {name: 'names', type: {type: 'array', items: 'string'}}
  ]
});

// A corresponding invalid record.
var invalidPerson = {age: null, names: ['ann', 3, 'bob']};
```

As a first use-case, we use the `errorHook` to implement a function to gather
all invalid paths a value (if any):

```javascript
function getInvalidPaths(type, val) {
  var paths = [];
  type.isValid(val, {errorHook: function (path) { paths.push(path.join()); }});
  return paths;
}

var paths = getInvalidPaths(personType, invalidPerson); // == ['age', 'names,1']
```

We can also implement an `assertValid` function which throws a helpful error on
the first mismatch encountered (if any):

```javascript
var util = require('util');

function assertValid(type, val) {
  return type.isValid(val, {errorHook: hook});

  function hook(path, obj) {
    throw new Error(util.format('invalid %s: %j', path.join(), obj));
  }
}

try {
  assertValid(personType, invalidPerson); // Will throw.
} catch (err) {
  // err.message === 'invalid age: null'
}
```

##### `type.clone(val, [opts])`

+ `val` {...} The object to copy.
+ `opts` {Object} Options:
  + `coerceBuffers` {Boolean} Allow coercion of JSON buffer representations
    into actual `Buffer` objects.
  + `fieldHook(field, obj, type)` {Function} Function called when each record
    field is populated. The value returned by this function will be used
    instead of `obj`. `field` is the current `Field` instance and `type` the
    parent type.

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

##### `type.random()`

Returns a random value of `type`.

##### `type.getName()`

Returns `type`'s fully qualified name if it exists, `undefined` otherwise.

##### `type.getSchema([noDeref])`

+ `noDeref` {Boolean} Do not dereference any type names.

Returns `type`'s canonical schema (as a string).

##### `type.getFingerprint([algorithm])`

+ `algorithm` {String} Algorithm used to generate the schema's [fingerprint][].
  Defaults to `md5`. In the browser, only `md5` is supported.

##### `Type.__reset(size)`

+ `size` {Number} New buffer size in bytes.

This method resizes the internal buffer used to encode all types. You should
only ever need to call this if you are encoding very large values and need to
reclaim memory.


#### Class `LongType(attrs, [opts])`

##### `LongType.using(methods, [noUnpack])`

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
  buffer with the long's unpacked representation. Otherwise, `toBuffer` should
  return an already packed buffer (of variable length).

+ `fromJSON(obj)`

  + `val` {Number|...} Parsed value. To ensure that the `fromString` method
    works correctly on data JSON-serialized according to the Avro spec, this
    method should at least support numbers as input.

  This method should return the corresponding decoded long.

  It might also be useful to support other kinds of input (typically the output
  of the long implementation's `toJSON` method) to enable serializing large
  numbers without loss of precision (at the cost of violating the Avro spec).

+ `toJSON(val)`

  + `val` {...} Decoded long.

  This method should return the long's JSON representation.

+ `isValid(val, [opts])`

  See [`Type.isValid`](#typeisvalidval-opts).

+ `compare(val1, val2)`

  See [`Type.compare`](#typecompareval1-val2).


#### Class `ArrayType(attrs, [opts])`

##### `type.getItemsType()`

The type of the array's items.


#### Class `EnumType(attrs, [opts])`

##### `type.getSymbols()`

Returs a copy of the type's symbols (an array of strings representing the
enum's valid values).

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.
Unlike the array returned by `getSymbols`, you can add, edit, and remove
aliases from this list.


#### Class `FixedType(attrs, [opts])`

##### `type.getSize()`

The size in bytes of instances of this type.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.
Unlike the array returned by `getSymbols`, you can add, edit, and remove
aliases from this list.


#### Class `MapType(attrs, [opts])`

##### `type.getValuesType()`

The type of the map's values (keys are always strings).


#### Class `RecordType(attrs, [opts])`

##### `type.getFields()`

Returns a copy of the array of fields contained in this record. Each field is
an object with the following methods:

+ `getAliases()`
+ `getDefault()`
+ `getName()`
+ `getOrder()`
+ `getType()`

##### `type.getRecordConstructor()`

The [`Record`](Api#class-record) constructor for instances of this type.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.
Unlike the array returned by `getFields`, you can add, edit, and remove aliases
from this list.


#### Class `UnionType(attrs, [opts])`

##### `type.getTypes()`

The possible types that this union can take.


#### Class `LogicalType(attrs, [opts,] [Types])`

"Abstract class" used to implement custom native types.

##### `type.getUnderlyingType()`

Get the underlying Avro type.

Implementors should override the following methods:

##### `type._fromValue(val)`

+ `val` {...}

##### `type._toValue(any)`

+ `any` {...}

##### `type._resolve(type)`

+ `type` {Type}


## Records

Each [`RecordType`](#class-recordtypeattrs-opts) generates a corresponding
`Record` constructor when its schema is parsed. It is available using the
`RecordType`'s `getRecordConstructor` methods. This helps make decoding and
encoding records more efficient.

All prototype methods below are prefixed with `$` to avoid clashing with an
existing record field (`$` is a valid idenfifier in JavaScript, but not in
Avro).

#### Class `Record(...)`

Calling the constructor directly can sometimes be a convenient shortcut to
instantiate new records of a given type.

##### `record.$clone([opts])`

Deep copy the record.

##### `record.$compare(val)`

Compare the record to another.

##### `record.$getType()`

Get the record's `type`.

##### `record.$isValid([opts])`

Check whether the record is valid.

##### `record.$toBuffer()`

Return binary encoding of record.

##### `record.$toString()`

Return JSON-stringified record.

##### `Record.getType()`

Convenience class method to get the record's type.


## Files and streams

*Not available in the browser.*

The following convenience functions are available for common operations on
container files:

#### `createFileDecoder(path, [opts])`

+ `path` {String} Path to Avro container file.
+ `opts` {Object} Decoding options, passed to
  [`BlockDecoder`](Api#class-blockdecoderopts).

Returns a readable stream of decoded objects from an Avro container file.

#### `createFileEncoder(path, schem, [opts])`

+ `path` {String} Destination path.
+ `schem` {Object|String|Type} Type used to serialize.
+ `opts` {Object} Encoding options, passed to
  [`BlockEncoder`](Api#class-blockencoderschem-opts).

Returns a writable stream of objects. These will end up serialized into an Avro
container file.

#### `extractFileHeader(path, [opts])`

+ `path` {String} Path to Avro container file.
+ `opts` {Object} Options:
  + `decode` {Boolean} Decode schema and codec metadata (otherwise they will be
    returned as bytes). Defaults to `true`.

Extract header from an Avro container file synchronously. If no header is
present (i.e. the path doesn't point to a valid Avro container file), `null` is
returned.


For more specific use-cases, the following stream classes are available in the
`avsc.streams` namespace:

+ [`BlockDecoder`](#blockdecoderopts)
+ [`RawDecoder`](#rawdecoderschem-opts)
+ [`BlockEncoder`](#blockencoderschem-opts)
+ [`RawEncoder`](#rawencoderschem-opts)


#### Class `BlockDecoder([opts])`

+ `opts` {Object} Decoding options. Available keys:
  + `codecs` {Object} Dictionary of decompression functions, keyed by codec
    name. A decompression function has the signature `fn(compressedData, cb)` where
    `compressedData` is a buffer of compressed data, and must call `cb(err,
    uncompressedData)` on completion. The default contains handlers for the
    `'null'` and `'deflate'` codecs.
  + `decode` {Boolean} Whether to decode records before returning them.
    Defaults to `true`.
  + `typeOpts` {Object} Options passed to instantiate the writer's `Type`.

A duplex stream which decodes bytes coming from on Avro object container file.

##### Event `'metadata'`

+ `type` {Type} The type used to write the file.
+ `codec` {String} The codec's name.
+ `header` {Object} The file's header, containing in particular the raw schema
  and codec.

##### Event `'data'`

+ `data` {Object|Buffer} Decoded element or raw bytes.

##### `BlockDecoder.getDefaultCodecs()`

Get built-in decompression functions (currently `null` and `deflate`).


#### Class `RawDecoder(schema, [opts])`

+ `schema` {Object|String|Type} Writer schema. Required since the input doesn't
  contain a header. Argument parsing logic is the same as for
  [`parse`](Api#parseschema-opts).
+ `opts` {Object} Decoding options. Available keys:
  + `decode` {Boolean} Whether to decode records before returning them.
    Defaults to `true`.

A duplex stream which can be used to decode a stream of serialized Avro objects
with no headers or blocks.

##### Event `'data'`

+ `data` {Object|Buffer} Decoded element or raw bytes.


#### Class `BlockEncoder(schema, [opts])`

+ `schema` {Object|String|Type} Schema used for encoding. Argument parsing
  logic is the same as for [`parse`](Api#parseschema-opts).
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


#### Class `RawEncoder(schema, [opts])`

+ `schema` {Object|String|Type} Schema used for encoding. Argument parsing
  logic is the same as for [`parse`](Api#parseschema-opts).
+ `opts` {Object} Encoding options. Available keys:
  + `batchSize` {Number} To increase performance, records are serialized in
    batches. Use this option to control how often batches are emitted. If it is
    too small to fit a single record, it will be increased automatically.
    Defaults to 64kB.

The encoding equivalent of `RawDecoder`.

##### Event `'data'`

+ `data` {Buffer} Serialized bytes.


[canonical-schema]: https://avro.apache.org/docs/current/spec.html#Parsing+Canonical+Form+for+Schemas
[schema-resolution]: https://avro.apache.org/docs/current/spec.html#Schema+Resolution
[sort-order]: https://avro.apache.org/docs/current/spec.html#order
[fingerprint]: https://avro.apache.org/docs/current/spec.html#Schema+Fingerprints
[custom-long]: Advanced-usage#custom-long-types
[logical-types]: Advanced-usage#logical-types
