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
  + A schema object (e.g. `{type: 'array', items: 'int'}`).
+ `opts` {Object} Parsing options forwarded to
  [`Type.fromSchema`](#typefromschemaobj-opts).

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
  + `LongType`, [`AbstractLongType`](#class-abstractlongtypeopts)
  + `NullType`
  + `StringType`
+ Complex types:
  + [`ArrayType`](#class-arraytypeschema-opts)
  + [`EnumType`](#class-enumtypeschema-opts)
  + [`FixedType`](#class-fixedtypeschema-opts)
  + [`MapType`](#class-maptypeschema-opts)
  + [`RecordType`](#class-recordtypeschema-opts)
  + [`UnionType`](#class-uniontypeschema-opts)


### Class `Type`

"Abstract" base Avro type class. All implementations inherit from it. Unless
specified otherwise, it is undefined behavior to override any of the methods
below.

##### `type.decode(buf, [pos,] [resolver])`

+ `buf` {Buffer} Buffer to read from.
+ `pos` {Number} Offset to start reading from.
+ `resolver` {Resolver} Optional resolver to decode records serialized from
  another schema. See [`createResolver`](#typecreateresolverwritertype) for how
  to create one.

Returns `{object: object, offset: offset}` if `buf` contains a valid encoding
of `type` (`object` being the decoded object, and `offset` the new offset in
the buffer). Returns `{object: undefined, offset: -1}` when the buffer is too
short.

##### `type.encode(obj, buf, [pos,] [noCheck])`

+ `obj` {Object} The object to encode.
+ `buf` {Buffer} Buffer to write to.
+ `pos` {Number} Offset to start writing at.
+ `noCheck` {Boolean} Do not check that the instance is valid before encoding
  it. Serializing invalid objects is undefined behavior, so use this only if
  you are sure the object satisfies the schema.

Encode an object into an existing buffer. If encoding was successful, returns
the new (non-negative) offset, otherwise returns `-N` where `N` is the
(positive) number of bytes by which the buffer was short.

##### `type.fromBuffer(buf, [resolver,] [noCheck])`

+ `buf` {Buffer} Bytes containing a serialized object of the correct type.
+ `resolver` {Resolver} To decode records serialized from another schema. See
  [`createResolver`](#typecreateresolverwritertype) for how to create an
  resolver.
+ `noCheck` {Boolean} Do not check that the entire buffer has been read. This
  can be useful when using an resolver which only decodes fields at the start of
  the buffer, allowing decoding to bail early.

Deserialize a buffer into its corresponding value.

##### `type.toBuffer(obj, [noCheck])`

+ `obj` {Object} The instance to encode. It must be of type `type`.
+ `noCheck` {Boolean} Do not check that the instance is valid before encoding
  it. Serializing invalid objects is undefined behavior, so use this only if
  you are sure the object satisfies the schema.

Returns a `Buffer` containing the Avro serialization of `obj`.

##### `type.createResolver(writerType)`

+ `writerType` {Type} Writer type.

Create a resolver that can be be passed to the `type`'s
[`decode`](#typedecodebuf-pos-resolver) and
[`fromBuffer`](#typefrombufferbuf-resolver-nocheck) methods. This will enable
decoding objects which had been serialized using `writerType`, according to the
Avro [resolution rules][schema-resolution]. If the schemas are incompatible,
this method will throw an error.

##### `type.fromString(str)`

+ `str` {String} String representing a JSON-serialized object.

Deserialize a JSON-encoded object of this type.

##### `type.toString([obj])`

+ `obj` {Object} The object to serialize. If not specified, this method will
  return the [canonical version][canonical-schema] of this type's schema
  instead (which can then be used to compare schemas for equality).

Serialize an object into a JSON-encoded string.

##### `type.random()`

Returns a random instance of this type.

*You can override this method to provide your own generator.*

##### `type.clone(obj, [opts])`

+ `obj` {Object} The object to copy.
+ `opts` {Object} Options:
  + `fieldHook(obj, field, type)` {Function} Function called when each record
    field is populated. The value returned by this function will be used
    instead of `obj`.
  + `coerceBuffers` {Boolean} Allow coercion of strings and JSON buffer
    representations into actual `Buffer` objects.
  + `wrapUnions` {Boolean} Wrap values corresponding to unions to the union's
    first type. This is to support encoding of field defaults as mandated by
    the spec (and should rarely come in useful otherwise).

Deep copy an object into a valid representation of `type`. An error will be
thrown if this is not possible.

##### `type.isValid(obj)`

+ `obj` {Object} The object to validate.

Check whether `obj` is a valid representation of `type`.

*You can override this method to support additional logic, but only to make it
stricter.*

##### `type.compare(obj1, obj2)`

+ `obj1` {Object} Instance of `type`.
+ `obj2` {Object} Instance of `type`.

Returns `0` if both objects are equal according to their [sort
order][sort-order], `-1` if the first is smaller than the second , and `1`
otherwise. Comparing invalid objects is undefined behavior.

##### `type.compareBuffers(buf1, buf2)`

+ `buf1` {Buffer} Buffer containing Avro encoding of an instance of `type`.
+ `buf2` {Buffer} Buffer containing Avro encoding of an instance of `type`.

Similar to [`compare`](#typecompareobj1-obj2), but doesn't require decoding
instances.

##### `Type.fromSchema(obj, [opts])`

+ `schema` {Object|String} A JavaScript object representing an Avro schema
  (e.g. `{type: 'array', items: 'int'}`). If a string is passed, it will be
  interpreted as a type name, to be looked up in the registry (see `opts`
  below).
+ `opts` {Object} Parsing options. The following keys are currently supported:
  + `namespace` {String} Optional parent namespace.
  + `registry` {Object} Optional registry of predefined type names. By default
    only Avro primitives have their names defined.
  + `typeHook(schema, opts)` {Function} Function called before each new type is
    instantiated. The relevant schema is available as first argument and the
    parsing options as second. This function can optionally return a type which
    will then be used in place of the result of parsing `schema`.

Parses a schema into its corresponding type.

##### `type.getFingerprint(algorithm)`

+ `algorithm` {String} Algorithm used to generate the schema's [fingerprint][].
  Defaults to `md5`. In the browser, only `md5` is supported.

##### `Type.__reset(size)`

+ `size` {Number} New buffer size in bytes.

This method resizes the internal buffer used to encode all types. You should
only ever need to call this if you are encoding very large objects and need to
reclaim memory.


#### Class `AbstractLongType([opts])`

+ `opts` {Object} Options.

  + `manualMode` {Boolean} Do not automatically unpack bytes before passing
    them to [`read`](#typereadbuf) and pack bytes returned by
    [`write`](#typewriteobj).

  Additionally, as a convenience, any keys matching the names of the methods
  below will be attached to `this` (similar to the [simplified stream
  API](https://nodejs.org/api/stream.html#stream_simplified_constructor_api)).
  This can be used to create a custom long type without inheritance.

This class provides an interface to support arbitrary long representations.
Doing so requires implementing the following methods (a couple examples are
also available [here][custom-long]):

##### `type.read(buf)`

+ `buf` {Buffer} Encoded long. If `manualMode` is off (the default), `buf` will
  be an 8-byte buffer containing the long's unpacked representation. Otherwise,
  `buf` will contain a variable length buffer with the long's packed
  representation.

This method should return the corresponding decoded long.

##### `type.write(obj)`

+ `obj` {...} Decoded long.

If `manualMode` is off (the default), this method should return an 8-byte
buffer with the long's unpacked representation. Otherwise, `write` should
return an already packed buffer (of variable length).

##### `type.fromJSON(obj)`

+ `obj` {Number|...} Parsed object. To ensure that the `fromString` method
  works correctly on data JSON-serialized according to the Avro spec, this
  method should at least support numbers as input.

This method should return the corresponding decoded long.

It might also be useful to support other kinds of input (typically the
output of the long's `toJSON` method) to enable serializing large numbers
without loss of precision (at the cost of violating the Avro spec).

##### `type.isValid(obj)`

See base `Type` method.

##### `type.compare(obj1, obj2)`

See base `Type` method.


#### Class `ArrayType(schema, [opts])`

##### `type.getItemsType()`

The type of the array's items.


#### Class `EnumType(schema, [opts])`

##### `type.getFullName()`

The type's fully qualified name.

##### `type.getSymbols()`

Returs a copy of the type's symbols (an array of strings representing the
enum's valid values).

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.
Unlike the array returned by `getSymbols`, you can add, edit, and remove
aliases from this list.


#### Class `FixedType(schema, [opts])`

##### `type.getFullName()`

The type's fully qualified name.

##### `type.getSize()`

The size in bytes of instances of this type.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.
Unlike the array returned by `getSymbols`, you can add, edit, and remove
aliases from this list.


#### Class `MapType(schema, [opts])`

##### `type.getValuesType()`

The type of the map's values (keys are always strings).


#### Class `RecordType(schema, [opts])`

##### `type.getFullName()`

The type's fully qualified name.

##### `type.getFields()`

Returns a copy of the array of fields contained in this record. Each field is
an object with the following methods:

+ `getAliases()`
+ `getDefault()`
+ `getName()`
+ `getOrder()`
+ `setOrder(order)`
+ `getType()`

##### `type.getRecordConstructor()`

The [`Record`](Api#class-record) constructor for instances of this type.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.
Unlike the array returned by `getFields`, you can add, edit, and remove aliases
from this list.


#### Class `UnionType(schema, [opts])`

##### `type.getTypes()`

The possible types that this union can take.


## Records

Each [`RecordType`](#class-recordtypeschema-opts) generates a corresponding
`Record` constructor when its schema is parsed. It is available using the
`RecordType`'s `getRecordConstructor` methods. This helps make decoding and
encoding records more efficient.

All prototype methods below are prefixed with `$` to avoid clashing with an
existing record field (`$` is a valid idenfifier in JavaScript, but not in
Avro).

#### Class `Record(...)`

Calling the constructor directly can sometimes be a convenient shortcut to
instantiate new records of a given type.

##### `record.$clone()`

Deep copy the record.

##### `record.$compare(obj)`

Compare the record to another.

##### `record.$getType()`

Get the record's `type`.

##### `record.$isValid()`

Check whether the record is valid.

##### `record.$toBuffer([noCheck])`

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

#### `createFileEncoder(path, type, [opts])`

+ `path` {String} Destination path.
+ `type` {Type} Type used to serialize.
+ `opts` {Object} Encoding options, passed to
  [`BlockEncoder`](Api#class-blockencodertype-opts).

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
+ [`RawDecoder`](#rawdecodertype-opts)
+ [`BlockEncoder`](#blockencodertype-opts)
+ [`RawEncoder`](#rawencodertype-opts)


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


#### Class `RawDecoder(type, [opts])`

+ `type` {Type} Writer type. Required since the input doesn't contain a header.
+ `opts` {Object} Decoding options. Available keys:
  + `decode` {Boolean} Whether to decode records before returning them.
    Defaults to `true`.

A duplex stream which can be used to decode a stream of serialized Avro objects
with no headers or blocks.

##### Event `'data'`

+ `data` {Object|Buffer} Decoded element or raw bytes.


#### Class `BlockEncoder(type, [opts])`

+ `type` {Type} The type to use for encoding.
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
  + `noCheck` {Boolean} Bypass record validation.
  + `omitHeader` {Boolean} Don't emit the header. This can be useful when
    appending to an existing container file. Defaults to `false`.
  + `syncMarker` {Buffer} 16 byte buffer to use as synchronization marker
    inside the file. If unspecified, a random value will be generated.

A duplex stream to create Avro container object files.

##### Event `'data'`

+ `data` {Buffer} Serialized bytes.


#### Class `RawEncoder(type, [opts])`

+ `type` {Type} The type to use for encoding.
+ `opts` {Object} Encoding options. Available keys:
  + `batchSize` {Number} To increase performance, records are serialized in
    batches. Use this option to control how often batches are emitted. If it is
    too small to fit a single record, it will be increased automatically.
    Defaults to 64kB.
  + `noCheck` {Boolean} Bypass record validation.

The encoding equivalent of `RawDecoder`.

##### Event `'data'`

+ `data` {Buffer} Serialized bytes.


[canonical-schema]: https://avro.apache.org/docs/current/spec.html#Parsing+Canonical+Form+for+Schemas
[schema-resolution]: https://avro.apache.org/docs/current/spec.html#Schema+Resolution
[sort-order]: https://avro.apache.org/docs/current/spec.html#order
[fingerprint]: https://avro.apache.org/docs/current/spec.html#Schema+Fingerprints
[custom-long]: Advanced-usage#custom-long-types
