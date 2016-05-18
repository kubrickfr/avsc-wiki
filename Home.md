+ [Overview](#what-is-avro)
+ [Browser support](#browser-support)

# Overview

`avsc` is a pure JavaScript implementation of the [Avro
specification][avro-specification].

## What is Avro?

Avro is a data serialization system. It is part of the [Apache Software
Foundation][asf] and is widely used both in offline systems (e.g. Hadoop) and
online (e.g. Kafka).

Quoting the [official documentation][avro-documentation]:

  Avro provides:

  + Rich data structures.
  + A compact, fast, binary data format.
  + A container file, to store persistent data.
  + Remote procedure call (RPC).
  + Simple integration with dynamic languages. Code generation is not required to
    read or write data files nor to use or implement RPC protocols. Code
    generation as an optional optimization, only worth implementing for
    statically typed languages.


# Browser usage

## Compatibility table

[![Browser compatibility table](https://saucelabs.com/browser-matrix/mtth.svg)](https://saucelabs.com/u/buffer)


## Distributions

Since not all of `avsc`'s functionality is always used and keeping library size
low is important for client-side libraries, `avsc` comes in multiple flavors.
Depending on which module you `require`, you will get a different distribution:

+ `'avsc/etc/browser/avsc'`: the full distribution (~51kB minified and
  compressed), including serialization, protocols, and Avro file support. This
  is also the default distribution you get when you `require('avsc')` directly.
+ `'avsc/etc/browser/avsc-prototols'`: a slightly lighter distribution (~34kB)
  which doesn't include file support.
+ `'avsc/etc/browser/avsc-types'`: the lightest distribution (~20kB) which
  only includes serialization support.

In all the above cases, the core `avsc` libraries only represent a fraction of
the total size (~15kB with everything included). If you were already using some
of `avsc`'s dependencies (e.g. `'events'`), your bundle will increase by less
than the sizes indicated.


[avro-specification]: https://avro.apache.org/docs/current/spec.html
[asf]: http://www.apache.org/
[avro-documentation]: http://avro.apache.org/docs/current/
