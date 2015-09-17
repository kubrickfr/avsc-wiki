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

Generate a random instance of this type.

##### `type.decode(buf)`

+ `buf` {Buffer} Bytes containing a serialized object of the correct type.

##### `type.encode(obj, [opts])`

Returns a `Buffer` containing the Avro serialization of `obj`.

+ `obj` {Object} The instance to encode. It must be of type `type`.
+ `opts` {Object} Encoding options. Currently available:
  + `size` {Number} The initial size of the buffer used to encode the object. Setting this appropriately will speed up encoding by reducing the number of resizes. Defaults to `1024`.
  + `unsafe` {Boolean} Do not check that the instance is valid before encoding it. This can yield a significant speed boost.

##### `type.isValid(obj)`

Check whether `obj` is a valid representation of `type`.

+ `obj` {Object} The object to validate.

##### `type.getTypeName()`

Get the type's name (e.g. `'int'`, `'record'`, ...).


#### Class `PrimitiveType(name)`

The common type used for `null`, `boolean`, `int`, `long`, `float`, `double`, `bytes`, and `string`.


#### Class `ArrayType(schema, [opts])`

##### `type.itemsType`

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

##### `type.valuesType`


#### Class `RecordType(schema, [opts])`

##### `type.name`
##### `type.doc`
##### `type.fields`

##### `type.getRecordConstructor()`

The `Record` constructor for instances of this type.

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

**Not yet implemented.**

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