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

Using the `typeHook` option, it is possible to introduce custom behavior on any
type. This can for example be used to override a type's `isValid` or `random`
method.

Below we show an example implementing a custom random float generator.

```javascript
var avsc = require('avsc');

/**
 * Hook which allows setting a range for float types.
 *
 * @param schema {Object} The type's corresponding schema.
 *
 */
var typeHook = function (schema) {
  var range = schema.range;
  if (this.type === 'float') {
    var span = range[1] - range[0];
    this.random = function () { return range[0] + span * Math.random(); };
  }
};

// We pass the above hook in the parsing options.
var type = avsc.parse({
  type: 'record',
  name: 'Recommendation',
  fields: [
    {name: 'id', type: 'long'},
    {name: 'score', type: {type: 'float', range: [-1, 1]}} // Note the range.
  ]
}, {typeHook: typeHook});
```


## Custom long types

JavaScript represents all numbers as doubles internally, which means than it is
possible to lose precision when using very large numbers (greater than
9,007,199,254,740,991 or smaller than a similar bound). For example:

```javascript
Number.parseInt('9007199254740995') === 9007199254740996 // true
```

In most cases, these bounds are so large that this is not a problem (timestamps
fit nicely inside the supported precision), however it might happen that the
full long range must be supported. (To avoid silently corrupting data, the
default `LongType` implementation will throw an error when encountering a
number outside the supported precision.)

JavaScript has several implementations of 64 bit integers, with different
characteristics (e.g. some are faster but do not run in the browser). Rather
than choose a particular one, `avsc` provides a generic
[`AbstractLongType`](Api#abstractlongtypeopts) which can be adapted to each.

Here are a few examples of using it with various implementations (refer to the
API for details on each option):

+ [`node-int64`](https://www.npmjs.com/package/node-int64)

  ```javascript
  var longType = new avsc.types.AbstractLongType({
    read: function (buf) { return new Int64(buf); },
    write: function (n) { return n.toBuffer(); },
    isValid: function (n) { return n instanceof Int64; },
    fromJSON: function (o) { return new Int64(o); },
    compare: function (n1, n2) { return n1.compare(n2); }
  });
  ```

+ [`long`](https://www.npmjs.com/package/long):

  ```javascript
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

Either of the above can then be used in place of the default `LongType` to
provide full 64 bit support when decoding and encoding Avro:

```javascript
var type = avsc.parse('./Schema.avsc', {registry: {'long': longType}});
```

Because the built-in JSON parser is itself limited by JavaScript's internal
number representation, using the `toString` and `fromString` methods is
generally still unsafe (see [`type.fromJSON`](Api#typefromjsonobj) for a
workaround).

Finally, to make integration easier, `read` (resp. `write`) expects an unpacked
(resp. packed) buffer. To leverage an external optimized packing and unpacking
routine (for example when using a native C++ addon), we can disable this
behavior by setting the `manualMode` constructor option to `true`.
