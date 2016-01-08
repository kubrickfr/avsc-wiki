# Browser support

[![Browser compatibility table](https://saucelabs.com/browser-matrix/mtth.svg)](https://saucelabs.com/u/buffer)

Since not all of `avsc`'s functionality is always used and keeping library size
low is important for client-side libraries, `avsc` comes in multiple flavors.
Depending on which module you `require`, you will get a different distribution:

+ `'avsc/etc/browser/avsc'`: the full distribution (~49kB minified and
  compressed), including serialization, protocols, and Avro file support. This
  is also the default distribution you get when you `require('avsc')` directly.
+ `'avsc/etc/browser/avsc-prototols'`: a slightly lighter distribution (~32kB)
  which doesn't include file support.
+ `'avsc/etc/browser/avsc-types'`: the lightest distribution (~20kB) which
  only includes serialization support.

In all the above cases, the core `avsc` libraries only represent a small
fraction of the total size (~15kB with everything included). If you were
already using some of `avsc`'s dependencies (e.g. `'events'`), your bundle will
increase by less than the sizes indicated.
