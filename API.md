# Parsing schemas

### `avsc.parse(schema, [opts])`

Parse a schema and return an instance of the corresponding `Type`.

+ `schema` {Object|String} Schema (type object or type name string).
+ `opts` {Object} Parsing options. The following keys are currently supported:
  + `namespace` {String} Optional parent namespace.
  + `registry` {Object} Optional registry of predefined type names.
  + `unwrapUnions` {Boolean} By default, Avro expects all unions to be wrapped inside an object with a single key. Setting this to `true` will prevent this, slightly improving performance (encoding is then done on the first type which validates).
  + `typeHook` {Function} Function called after each new Avro type is instantiated. The new type is available as `this` and the relevant schema as first and only argument.

### `avsc.parseFile(path, [opts])`

Convenience function to parse a schema file directly.

+ `path` {String} Path to schema file.
+ `opts` {Object} Parsing options (identical to those of `parse`).


# Avro types

Serializing a `type` back to JSON (e.g. using `JSON.stringify`) will return a valid equivalent Avro schema!

It is also possible to generate types programmatically, using the classes below. They are all available in the `avsc.types` namespace.


### Class `Type`

"Abstract" base Avro type class. All implementations (see below) have the following property and methods:

##### `type.type`

The type's name (e.g. `'int'`, `'record'`, ...).

##### `type.random()`

Generate a random instance of this type.

##### `type.isValid(obj)`

Check whether `obj` is a valid representation of `type`.

+ `obj` {Object} The object to validate.

##### `type.encode(obj, [opts])`

Returns a `Buffer` containing the Avro serialization of `obj`.

+ `obj` {Object} The instance to encode. It must be of type `type`.
+ `opts` {Object} Encoding options. Currently available:
  + `buffer`, used to serialize the object into (a slice will be returned). If not passed, or if the serialized object doesn't fit into the passed buffer, a new one will be created.
  + `unsafe` {Boolean} Do not check that the instance is valid before encoding it. This can yield a significant speed boost.

##### `type.decode(buf, [adapter])`

+ `buf` {Buffer} Bytes containing a serialized object of the correct type.
+ `adapter` {Adapter} To read records serialized using another schema. See `createAdapter`.

##### `type.createAdapter(writerType)`

+ `writerType` {Type} Writer type.


#### Class `PrimitiveType(name)`

The common type used for `null`, `boolean`, `int`, `long`, `float`, `double`, `bytes`, and `string`.


#### Class `ArrayType(schema, [opts])`

##### `type.items`

The `type` of the array's items.


#### Class `EnumType(schema, [opts])`

##### `type.name`
##### `type.doc`
##### `type.symbols`

The enum's name, documentation, and symbols list.

Instances of this type will either be represented as wrapped objects (according to the Avro spec), or as their value directly (if `unwrapUnions` was set when parsing the schema).


#### Class `FixedType(schema, [opts])`

##### `type.name`
##### `type.size`

Instances of this type will be `Buffer`s.


#### Class `MapType(schema, [opts])`

##### `type.values`


#### Class `RecordType(schema, [opts])`

##### `type.name`
##### `type.doc`
##### `type.fields`

##### `type.getRecordConstructor()`

The `Record` constructor for instances of this type.

##### `type.asReaderOf(writerType)`

Returns a type suitable for reading a file written using a different schema.

+ `writerType` {Type} A compatible `type`.


#### Class `UnionType(schema, [opts])`

##### `type.types`


# Records

### Class `Record(...)`

Specific record class, programmatically generated for each record schema.

#### `Record.random()`
#### `Record.decode(buf, [adapter])`
#### `record.$encode([opts])`
#### `record.$isValid()`
#### `record.$type`


# Reading and writing files

**Not yet implemented.**

### `avsc.createReadStream(path, [opts])`

+ `path` {String}
+ `opts` {Object} Decoding options. Available keys:
  + `containerFile` {Boolean} By default the stream will try to infer whether the input comes from a container file or not by looking at the first four bytes (depending on whether they match Avro's magic bytes or not). This option can be used to explicitly enforce this.
  + `readerType` {AvroType} Required when reading a non-container file. When reading a container file, this will be used to generate a reader type adapted to the writer's schema.
  + `includeBuffer` {Boolean}


### Class `Decoder([opts])`

+ `opts` {Object} Decoding options. Available keys:
  + `readerType` {AvroType} Reader type.
  + `includeBuffer` {Boolean}

#### `getReaderType()`

The type used to read the file.

#### `getWriterType()`

The type used to write the file.

#### Event `'metadata'`

+ `meta` {Object} The header's metadata, containing the raw schema and codec.
+ `sync` {Buffer} Sync marker for the file.

#### Event `'data'`

+ `data` {...} Decoded element.
+ `buf` {Buffer} (Only if `includeBuffer` is set.)


### Class `avsc.Encoder([opts])`

+ `opts` {Object} Encoding options. Available keys:
  + `writerType` {AvroType} Writer type. As a convenience, this will be inferred if writing `Record` instances (from the first one passed).
  + `codec` {String}
  + `blockSize` {Number}

#### `getWriterType()`

Get the type used to serialize the records.

#### Event `'data'`

+ `data` {Buffer} Encoded block.