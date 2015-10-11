+ [Schema evolution](#schema-evolution)
+ [Type hooks](#type-hooks)
+ [Unwrapping unions](#unwrapping-unions)


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
var resolver = v2.parse(v1);

// And pass it whenever we want to decode from the previous version.
var buf = v1.encode({name: 'Ann', age: 25}); // Encode using old schema.
var obj = v2.decode(buf, resolver); // === {name: {string: 'Ann'}, phone: null}
```

Reader's schemas are also very useful for performance, letting us decode only
fields that are needed.


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

## Unwrapping unions

The Avro specification mandates the use of single-key maps to represent decoded
union values (except `null`, which is never wrapped). The map's unique key is
the name of the selected type . So the string `Hi!` would be represented by
`{string: 'Hi!'}` for the union schema `["null", "string"]`.

This makes serializing decoded values unambiguous (union schemas disallow
multiple types with the same name), but is overkill in most cases (e.g. when
unions are only used to make a field nullable). For this reason, `avsc`
provides an `unwrapUnions` option, which will decode unions directly into their
value.

For example:

```javascript
var schema = ['null', 'string'];
var buf = new Buffer([2, 6, 48, 69, 21]);

var wrappedType = avsc.parse(schema);
wrappedType.decode(buf); // === {string: 'Hi!'}

var unwrappedType = avsc.parse(schema, {unwrapUnions: true});
unwrappedType.decode(buf); // === 'Hi!'
```

When the types inside a union are unambiguous, this option can greatly simplify
union-heavy schemas. It also provides a performance boost to decoding (at a
small cost to encoding).
