# API

## Parsing

### `avsc.parse(schema, [opts])`

Parse a schema and return an instance of the corresponding `Type`.

+ `schema` {Object|String} Schema (type object or type name string).
+ `opts` {Object} Parsing options. The following keys are currently supported:

  + `namespace` {String} Optional parent namespace.
  + `registry` {Object} Optional registry of predefined type names.
  + `unwrapUnions` {Boolean} By default, Avro expects all unions to be wrapped
    inside an object with a single key. Setting this to `true` will prevent
    this.

### `avsc.parseFile(path, [opts])`

Convenience function to parse a schema file.

+ `path` {String} Path to schema file.
+ `opts` {Object} Parsing options. See `parse` above for details.


## Avro types

It is also possible to generate types programmatically, using the classes below.

### `class Type`

"Abstract" base Avro type class. All implementations (see below) have the
following methods:

##### `type.random()`
##### `type.decode(buf)`
##### `type.encode(obj, [opts])`
##### `type.isValid(obj)`
##### `type.getTypeName()`

Implementations:

#### `class ArrayType(schema, [opts])`
##### `type.itemsType`

#### `class EnumType(schema, [opts])`
##### `type.name`
##### `type.doc`
##### `type.symbols`

#### `class FixedType(schema, [opts])`
##### `type.name`
##### `type.size`

#### `class MapType(schema, [opts])`
##### `type.valuesType`

#### `class PrimitiveType(name)`

#### `class RecordType(schema, [opts])`
##### `type.name`
##### `type.doc`
##### `type.fields`
##### `type.getRecordConstructor()`

#### `class UnionType(schema, [opts])`
##### `type.types`

### `class Record(...)`

Specific record class, programmatically generated for each record schema.

##### `Record.random()`
##### `Record.decode(buf)`
##### `record.$encode([opts])`
##### `record.$isValid()`
##### `record.$type`

