+ [Schema evolution](#schema-evolution)
+ [Type hooks](#type-hooks)


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
