+ [Schema evolution](#schema-evolution)
+ [Type hooks](#type-hooks)
+ [Custom long types](#custom-long-types)


## Schema evolution

Avro supports reading data written by another schema (as long as the reader's
and writer's schemas are compatible). We can do this by creating an appropriate
`Resolver`:

```javascript
var avsc = require('avsc');

// A schema's first version.
var v1 = avsc.parse({
  type: 'record',
  name: 'Person',
  fields: [
    {name: 'name', type: 'string'},
    {name: 'age', type: 'int'}
  ]
});

// The updated version.
var v2 = avsc.parse({
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

// We instantiate the resolver once.
var resolver = v2.createResolver(v1);

// And pass it whenever we want to decode from the previous version.
var buf = v1.toBuffer({name: 'Ann', age: 25}); // Encode using old schema.
var obj = v2.fromBuffer(buf, resolver); // === {name: {string: 'Ann'}, phone: null}
```

Reader's schemas are also very useful for performance, only decoding fields
that are needed.


## Type hooks

Using the `typeHook` option when parsing a schema, it is possible to introduce
custom behavior on any type. This can for example be used to override a type's
`random` method.

Below we show an example implementing a custom random float generator:

```javascript
var avsc = require('avsc');

var type = avsc.parse({
  type: 'record',
  name: 'Recommendation',
  fields: [
    {name: 'id', type: 'long'},
    {name: 'score', type: {type: 'float', range: [-1, 1]}}
  ]
}, {typeHook: typeHook}); // Note the hook.

/**
 * Hook which allows setting a range for float types.
 *
 * @param schema {Object} A type's schema.
 *
 */
function typeHook(schema) {
  if (schema.type !== 'float') {
    return;
  }

  var range = schema.range || [0, 1];
  var span = range[1] - range[0];
  var type = new avsc.types.FloatType();
  type.random = function () { return range[0] + span * Math.random(); };
  return type;
}
```


## Custom long types

JavaScript represents all numbers as doubles internally, which means than it is
possible to lose precision when using very large numbers (absolute value
greater than `9e+15` or so). For example:

```javascript
Number.parseInt('9007199254740995') === 9007199254740996 // true
```

In most cases, these bounds are so large that this is not a problem (timestamps
fit nicely inside the supported precision). However it might happen that the
full long range must be supported. For this reason, `avsc` lets us define
custom long implementations. (To avoid silently corrupting data, the default
`LongType` will throw an error when encountering a number outside the supported
precision range.)

There are multiple JavaScript libraries to represent 64 bit integers, with
different characteristics (e.g. some are faster but do not run in the browser).
Rather than choose a particular one, `avsc` provides a generic
[`AbstractLongType`](Api#abstractlongtypeopts) which can be adapted to each.
Below are a few implementation examples (refer to the API documentation for
details on each option; a helper script is also available to validate your
implementation inside `etc/scripts/`):

+ [`node-int64`](https://www.npmjs.com/package/node-int64):

  ```javascript
  var Int64 = require('node-int64');

  var longType = new avsc.types.AbstractLongType({
    read: function (buf) { return new Int64(buf); },
    write: function (n) { return n.toBuffer(); },
    isValid: function (n) { return n instanceof Int64; },
    fromJSON: function (o) { return new Int64(o); },
    compare: function (n1, n2) { return n1.compare(n2); }
  });
  ```

+ [`int64-native`](https://www.npmjs.com/package/int64-native):

  ```javascript
  var Int64Native = require('int64-native');

  var longType = new avsc.types.AbstractLongType({
    read: function (buf) { return new Int64Native('0x' + buf.toString('hex')); },
    write: function (n) { return new Buffer(n.toString().slice(2), 'hex'); },
    isValid: function (n) { return n instanceof Int64Native; },
    fromJSON: function (o) { return new Int64Native(o); },
    compare: function (n1, n2) { return n1.compare(n2); }
  });
  ```

+ [`long`](https://www.npmjs.com/package/long):

  ```javascript
  var Long = require('long');

  var longType = new avsc.types.AbstractLongType({
    read: function (buf) {
      return new Long(buf.readInt32LE(), buf.readInt32LE(4));
    },
    write: function (n) {
      var buf = new Buffer(8);
      buf.writeInt32LE(n.getLowBits());
      buf.writeInt32LE(n.getHighBits(), 4);
      return buf;
    },
    isValid: Long.isLong,
    fromJSON: Long.fromValue,
    compare: Long.compare
  });
  ```

Any such implementation can then be used in place of the default `LongType` to
provide full 64 bit support when decoding and encoding binary data from any
schema. To do so, override the default type used for `long`s by adding your
implementation to the `registry`:

```javascript
// Our schema here is very simple, but this would work for arbitrarily complex
// ones (applying to all longs inside of it).
var type = avsc.parse('long', {registry: {'long': longType}});

// Avro serialization of Number.MAX_SAFE_INTEGER + 4 (which is incorrectly
// rounded when represented as a double):
var buf = new Buffer([0x86, 0x80, 0x80, 0x80, 0x80, 0x80, 0x80, 0x20]);

// Assuming we are using the `node-int64` implementation.
var obj = new Int64(buf);
var encoded = type.toBuffer(obj); // == buf
var decoded = type.fromBuffer(buf); // == obj (No precision loss.)
```

Because the built-in JSON parser is itself limited by JavaScript's internal
number representation, using the `toString` and `fromString` methods is
generally still unsafe (see [`type.fromJSON`](Api#typefromjsonobj) for a
workaround).

Finally, to make integration easier, `read` and `write` deal with already
unpacked buffers by default. To leverage an external optimized packing and
unpacking routine (for example when using a native C++ addon), we can disable
this behavior by setting the `manualMode` constructor option to `true`.
