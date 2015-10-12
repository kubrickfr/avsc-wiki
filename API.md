+ [Parsing schemas](#parsing-schemas)
+ [Avro types](#avro-types)
+ [Records](#records)
+ [Streams](#streams)


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
  + `LongType`
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

"Abstract" base Avro type class. All implementations inherit from it.


##### `type.random()`

Returns a random instance of this type.


##### `type.clone(obj, [opts])`

+ `obj` {Object} The object to copy.
+ `opts` {Object} Options:
  + `fieldHook(obj, recordType)` {Function} Function called when each record
    field is instantiated. The field will be available as `this`. The value
    returned by this function will be used instead of `obj`.
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


##### `type.decode(buf, [pos,] [resolver])`

+ `buf` {Buffer} Buffer to read from.
+ `pos` {Number} Offset.
+ `resolver` {Resolver} Optional resolver.

For decoding many objects, prefer the use of decoding streams. Returns {obj,
n}.


##### `type.encode(obj, buf, [pos,] [noCheck])`

+ `obj` {Object} The object to encode.
+ `buf` {Buffer} Buffer to write to.
+ `pos` {Number} Offset.
+ `noCheck` {Boolean}

For encoding many objects, prefer the use of encoding streams. Returns the
number of bytes written.


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
[`decode`](#typefrombufferbuf-resolver-nocheck) method. This will enable decoding
objects which had been serialized using `writerType`, according to the Avro
[resolution rules][schema-resolution]. If the schemas are incompatible, this
method will throw an error.


##### `type.fromString(str)`

+ `str` {String} String representing a JSON-serialized object.

Deserialize a JSON-encoded object of this type.


##### `type.toString([obj])`

+ `obj` {Object} The object to serialize. If not specified, this method will
  return the [canonical version][canonical-schema] of this type's schema
  instead (which can then be used to compare schemas for equality).

Serialize an object into a JSON-encoded string.


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


##### `Type.fromSchema(schema, [opts])`

+ `schema` {Object|String} A JavaScript object representing an Avro schema
  (e.g. `{type: 'array', items: 'int'}`). If a string is passed, it will be
  interpreted as a type name, to be looked up in the registry (see `opts`
  below).
+ `opts` {Object} Parsing options. The following keys are currently supported:
  + `namespace` {String} Optional parent namespace.
  + `registry` {Object} Optional registry of predefined type names.
  + `typeHook(schema)` {Function} Function called after each new Avro type is
    instantiated. The new type is available as `this` and the relevant schema
    as first and only argument.

Parses a schema into its corresponding type.


##### `Type.getDefaultRegistry()`

Returns a copy of the default registry used to look up type names. It contains
the names of all Avro primitives. This is useful to prime a registry to be
passed to [`Type.fromSchema`](#typefromschemaobj-opts).


##### `type.getFingerprint(algorithm)`

+ `algorithm` {String} Algorithm used to generate the schema's [fingerprint][].
  Defaults to `md5`.


##### `Type.__reset(size)`

+ `size` {Number} New buffer size in bytes.

This method resizes the internal buffer used to encode all types. You should
only ever need to call this if you are encoding very large objects and need to
reclaim memory.


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

The `Record` constructor for instances of this type.

##### `type.getAliases()`

Optional type aliases. These are used when adapting a schema from another type.
Unlike the array returned by `getSymbols`, you can add, edit, and remove
aliases from this list.


#### Class `UnionType(schema, [opts])`

##### `type.getTypes()`

The possible types that this union can take.


## Records

Each [`RecordType`](#class-recordtype-opts) generates a corresponding `Record`
constructor when its schema is parsed. It is available using the `RecordType`'s
`getRecordConstructor` methods. This makes decoding records more efficient and
lets us provide the following convenience methods:

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


## Streams

The following functions are available for common operations on container files:

### `createFileDecoder(path, [opts])`

+ `path` {String} Path to Avro container file.
+ `opts` {Object} Decoding options, passed to `BlockDecoder`.

Returns a readable stream of decoded objects from an Avro container file.


### `extractFileHeader(path, [opts])`

+ `path` {String} Path to Avro container file.
+ `opts` {Object} Options:
  + `decode` {Boolean} Decode schema and codec metadata (otherwise they will be
    returned as bytes). Defaults to true.

Extract header from an Avro container file synchronously. If no header is
present (i.e. the path doesn't point to a valid Avro container file), `null` is
returned.


The following stream classes are available in the `avsc.streams` namespace:

+ [`RawDecoder`](#rawdecodertype-opts)
+ [`FrameDecoder`](#framedecodertype-opts)
+ [`BlockDecoder`](#blockdecoderopts)
+ [`RawEncoder`](#rawencodertype-opts)
+ [`FrameEncoder`](#frameencodertype-opts)
+ [`BlockEncoder`](#blockencodertype-opts)


### Class `RawDecoder(type, [opts])`

+ `type` {Type} Writer type. Required since the input doesn't contain a header.
+ `opts` {Object} Decoding options. Available keys:
  + `decode` {Boolean} Whether to decode records before returning them.
    Defaults to `true`.

A duplex stream which can be used to decode a stream of serialized Avro objects
with no headers or blocks.

#### Event `'data'`

+ `data` {Object|Buffer} Decoded element or raw bytes.


### Class `FrameDecoder(type, [opts])`

+ `type` {Type} Writer type. Required since the input doesn't contain a header.
+ `opts` {Object} Decoding options. Available keys:
  + `decode` {Boolean} Whether to decode records before returning them.
    Defaults to `true`.

A duplex stream which can be used to decode a stream of serialized Avro objects
with no headers or blocks.

#### Event `'data'`

+ `data` {Object|Buffer} Decoded element or raw bytes.


### Class `BlockDecoder([opts])`

+ `opts` {Object} Decoding options. Available keys:
  + `decode` {Boolean} Whether to decode records before returning them.
    Defaults to `true`.
  + `typeOpts` {Object} Options passed to instantiate the writer's `Type`.

A duplex stream which decodes bytes coming from on Avro object container file.

#### Event `'metadata'`

+ `type` {Type} The type used to write the file.
+ `codec` {String} The codec's name.
+ `header` {Object} The file's header, containing in particular the raw schema
  and codec.

#### Event `'data'`

+ `data` {Object|Buffer} Decoded element or raw bytes.


### Class `RawEncoder(type, [opts])`

+ `type` {Type} The type to use for encoding.
+ `opts` {Object} Encoding options. Available keys:
  + `batchSize` {Number} To increase performance, records are serialized in
    batches. Use this option to control how often batches are emitted. If it is
    too small to fit a single record, it will be increased automatically.
    Defaults to 64kB.
  + `noCheck` {Boolean} Whether to check each record before encoding it.
    Defaults to `true`.

The encoding equivalent of `RawDecoder`.

#### Event `'data'`

+ `data` {Buffer} Serialized bytes.


### Class `FrameEncoder(type, [opts])`

+ `type` {Type} The type to use for encoding.
+ `opts` {Object} Encoding options. Available keys:
  + `frameSize` {Number} Frame size in bytes. Defaults to 1024. Currently only
    one record is encoded per frame.
  + `noCheck` {Boolean} Whether to check each record before encoding it.
    Defaults to `true`.

The encoding equivalent of `RawDecoder`.

#### Event `'data'`

+ `data` {Buffer} Serialized bytes.


### Class `BlockEncoder(type, [opts])`

+ `type` {Type} The type to use for encoding.
+ `opts` {Object} Encoding options. Available keys:
  + `codec` {String} Name of codec to use for encoding.
  + `blockSize` {Number} Maximum uncompressed size of each block data. A new
    block will be started when this number is exceeded. If it is too small to
    fit a single element, it will be increased appropriately. Defaults to 64kB.
  + `omitHeader` {Boolean} Don't emit the header. This can be useful when
    appending to an existing container file. Defaults to `false`.
  + `syncMarker` {Buffer} 16 byte buffer to use as synchronization marker
    inside the file. If unspecified, a random value will be generated.
  + `noCheck` {Boolean} Whether to check each record before encoding it.
    Defaults to `true`.

A duplex stream to create Avro container object files.

#### Event `'data'`

+ `data` {Buffer} Serialized bytes.


[canonical-schema]: https://avro.apache.org/docs/current/spec.html#Parsing+Canonical+Form+for+Schemas
[schema-resolution]: https://avro.apache.org/docs/current/spec.html#Schema+Resolution
[sort-order]: https://avro.apache.org/docs/current/spec.html#order
[fingerprint]: https://avro.apache.org/docs/current/spec.html#Schema+Fingerprints
