<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Types](#types)
  - [What is a `Type`?](#what-is-a-type)
  - [How do I get a `Type`?](#how-do-i-get-a-type)
  - [What about Avro files?](#what-about-avro-files)
- [Services](#services)
  - [Defining a `Service`](#defining-a-service)
  - [Server implementation](#server-implementation)
  - [Calling our service](#calling-our-service)
- [Next steps](#next-steps)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Types

One of the main features provided by [Avro][] is a way to encode (_serialize_)
data. Once data is encoded, it can be stored in a file or a database, sent
across the internet to be decoded on another computer (_deserialized_), etc.

Many different encodings exist. [JSON][] for example is very commonly used from
JavaScript: it's built-in (via `JSON.parse` and `JSON.stringify`), reasonably
fast, and produces human-readable encodings.

```javascript
> pet = {kind: 'DOG', name: 'Beethoven', age: 4};
> str = JSON.stringify(pet);
'{"kind":"DOG","name":"Beethoven","age":4}' // Encoded data (still readable).
> str.length
41 // Number of bytes in the encoding.
```

JSON isn't always the most adequate encoding though:

+ It produces relatively large encodings since the keys (`kind`, `name`, and
  `age` above) are repeated in the output.
+ It doesn't enforce any properties on the data, so any validation has to be
  done separately.

Avro `type`s provide an alternate serialization mechanism, with a different set
of properties:

+ Schema-aware.
+ Binary, compact, faster.


## What is a `Type`?

A `type` is an JavaScript object which knows how to
[`decode`](Api#typedecodebuf-pos-resolver) and
[`encode`](Api#typeencodeval-buf-pos) a "family" of values. Examples of
supported families include:

+ All strings.
+ All arrays of numbers.
+ All `Buffer`s of length 4.
+ All objects with an integer `id` property and string `name` property.

Writing a schema can be daunting...

```javascript
> avro = require('avsc');
> inferredType = avro.Type.forValue(pet);
> inferredType.schema();
{ type: 'record', // "Record" is Avro parlance for "structured object".
  fields:
   [ { name: 'kind', type: 'string' }, // Each field corresponds to a property.
     { name: 'name', type: 'string' },
     { name: 'age', type: 'int' } ] }
```

TODO...

```javascript
> buf = inferredType.toBuffer(pet);
> buf.length
15 // 60% smaller than JSON!
```

```javascript
> inferredType.isValid({kind: 'CAT', name: 'Garfield', age: 5.2});
false // The age isn't an integer.
> inferredType.isValid({name: 'Mozart', age: 3});
false // The kind field is missing.
> inferredType.isValid({kind: 'PIG', name: 'Babe', age: 2});
true // All fields match.
```

```javascript
> exactType = avro.Type.forSchema({
...   type: 'record',
...   fields: [
...     {name: 'kind', type: {type: 'enum', symbols: ['CAT', 'DOG']}},
...     {name: 'name', type: 'string'},
...     {name: 'age', type: 'int'}
...   ]
... });
> buf = exactType.toBuffer(pet);
> buf.length
12 // 70% smaller than JSON!
```

```javascript
> exactType.isValid({kind: 'PIG', name: 'Babe', age: 2});
false // The pig kind wasn't defined in our enum.
> exactType.isValid({kind: 'DOG', name: 'Lassie', age: 5});
true // But dog was.
```


# Services

As well as defining an encoding, Avro defines a standard way of executing
remote procedure calls (_RPCs_), exposed via _services_. By leveraging these
services, we can implement portable and "type-safe" APIs:

+ Clients and servers can be implemented once and reused over many different
  transports (in-memory, TCP, HTTP, etc.).
+ All data flowing through the API is automatically validated: call arguments
  and return values are guaranteed to match the types specified in the API.

In this section, we'll walk through an example of building a simple link
management service similar to [bitly][].

## Defining a `Service`

The first step to creating a service is to define its _protocol_, describing
the available API calls and their signature. There are a couple ways of doing
so; we can write the [JSON declaration][json-protocol] directly, or we can use
Avro's [IDL syntax][idl] (which can then be compiled to JSON). The latter is
typically more convenient so we will use this here.

```java
/** A simple service to shorten URLs. */
protocol LinkService {

  /** Map a URL to an alias, throwing an error if it already exists. */
  null createAlias(string alias, string url);

  /** Expand an alias, returning null if the alias doesn't exist. */
  union { null, string } expandAlias(string alias);
}
```

With the above spec saved to a file, say `LinkService.avdl`, we can instantiate
the corresponding service as follows:

```javascript
// We first compile the IDL specification into a JSON protocol.
avro.assembleProtocol('./LinkService.avdl', function (err, protocol) {
  // From which we can create our service.
  const service = avro.Service.fromProtocol(protocol);
});
```

The `service` object can then be used generate clients and servers, as
described in the following sections.

## Server implementation

So far, we haven't said anything about how our API's responses will be
computed. This is where servers come in, servers provide the logic powering our
API.

For each call declared in the protocol (`createAlias` and `expandAlias` above),
servers expose a similarly named handler (`onCreateAlias` and `onExpandAlias`)
with the same signature:

```javascript
const urlCache = new Map(); // We'll use an in-memory map to store links.

// We instantiate a server corresponding to our API and implement both calls.
const server = service.createServer()
  .onCreateAlias(function (alias, url, cb) {
    if (urlCache.has(alias)) {
      cb(new Error('alias already exists'));
    } else {
      urlCache.set(alias, url); // Add the mapping to the cache.
      cb();
    }
  })
  .onExpandAlias(function (alias, cb) {
    cb(null, urlCache.get(alias));
  });
```

Notice that no part of the above implementation is coupled to a particular
communication scheme (e.g. HTTP, TCP, AMQP): the code we wrote is
_transport-agnostic_.

## Calling our service

The simplest way to call our service is use an in-memory client, passing in our
`server` above as option to `service.createClient`:

```javascript
const client = service.createClient({server});

// We first send a request to create an alias.
client.createAlias('hn', 'https://news.ycombinator.com/', function (err) {
  // Which we can now expand.
  client.expandAlias('hn', function (err, url) {
    console.log(`hn is currently aliased to ${url}`);
  });
});
```

The above is handy for local testing or quick debugging. More interesting
perhaps is the ability to communicate with our server over any binary streams,
for example TCP sockets:

```javascript
const net = require('net');

// Set up the server to listen to incoming connections on port 24950.
net.createServer()
  .on('connection', function (con) { server.createChannel(con); })
  .listen(24950);

// And create a matching client:
const client = service.createClient({transport: net.connect(24950)});
```

Note that RPC calls messages are always sent asynchronously and in parallel:
requests do not block each other. Furthermore, responses are available as soon
as they are received from the server; the client keeps track of which calls are
pending and triggers the right callbacks as responses come back.

Both above transports (in-memory and TCP) have the additional property of being
[_stateful_][transports]: each connection can be used to exchange multiple
messages, making them particularly efficient (avoiding the overhead of
handshakes). These aren't the only kind though, it is possible to exchange
messages over stateless connections, for example HTTP:

```javascript
const http = require('http');

// Each HTTP request/response will correspond to a single API call.
http.createServer()
  .on('request', function (req, res) {
    server.createChannel(function (cb) { cb(null, res); return req; });
  })
  .listen(8080);

// Similarly, an HTTP client:
const client = service.createClient({transport: function (cb) {
  return http.request({method: 'POST', port: 8080})
    .on('response', function (res) { cb(null, err); })
    .on('error', cb);
}});
```


# Next steps

The [API documentation](Api) provides a comprehensive list of available
functions and their options. The [Advanced usage section](Advanced-usage) goes
through a few more examples of advanced functionality.


[avro]: https://avro.apache.org/docs/1.8.0/index.html
[json]: http://www.json.org/
[bitly]: https://bitly.com/
[json-protocol]: https://avro.apache.org/docs/1.8.0/spec.html#Protocol+Declaration
[transports]: https://avro.apache.org/docs/1.8.0/spec.html#Message+Transport
[idl]: https://avro.apache.org/docs/1.8.0/idl.html
[object-container]: https://avro.apache.org/docs/current/spec.html#Object+Container+Files
[rstream]: https://nodejs.org/api/stream.html#stream_class_stream_readable
