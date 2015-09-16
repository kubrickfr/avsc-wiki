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

It is also possible to generate types programmatically, using the classes below. They are all available in the `avsc.types` namespace.

### Class `Type`

"Abstract" base Avro type class. All implementations (see below) have the following methods:

##### `type.random()`
##### `type.decode(buf)`
##### `type.encode(obj, [opts])`
##### `type.isValid(obj)`
##### `type.getTypeName()`

Implementations:

#### Class `ArrayType(schema, [opts])`
##### `type.itemsType`

#### Class `EnumType(schema, [opts])`
##### `type.name`
##### `type.doc`
##### `type.symbols`

#### Class `FixedType(schema, [opts])`
##### `type.name`
##### `type.size`

#### Class `MapType(schema, [opts])`
##### `type.valuesType`

#### Class `PrimitiveType(name)`

#### Class `RecordType(schema, [opts])`
##### `type.name`
##### `type.doc`
##### `type.fields`
##### `type.getRecordConstructor()`

#### Class `UnionType(schema, [opts])`
##### `type.types`

# Records

### Class `Record(...)`

Specific record class, programmatically generated for each record schema.

#### `Record.random()`
#### `Record.decode(buf)`
#### `record.$encode([opts])`
#### `record.$isValid()`
#### `record.$type`

# Reading and writing files

The following streams are available:

### Class `avsc.Decoder([opts])`

+ `opts` {Object} Decoding options. Available keys:
  + `containerFile` {Boolean} By default the stream will try to infer whether the input comes from a container file by looking at the first four bytes (depending on whether they match Avro's magic bytes or not). This option can be used to explicitly enforce this.
  + `type` {AvroType} Required when reading a non-container file. When reading a container file, this will be used as reader type.

#### Event `'metadata'`

+ `schema` {Object} The context will be set to the stream itself. To override the writer type, set the `writerType` property inside the callback.

#### Event `'data'`

+ `data` Decoded element.

### Class `avsc.Encoder([opts])`

+ `opts` {Object} Encoding options. Available keys:
  + `containerFile` {Boolean} Defaults to `true`.
  + `type` {AvroType} Inferred if writing `Record` instances.
  + `codec` {String}
  + `blockSize` {Number}

#### Event `'data'`

+ `data` {Buffer} Encoded block (if `containerFile` above is `true`) or element (otherwise).