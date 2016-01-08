`avsc`

[![Browser compatibility table](https://saucelabs.com/browser-matrix/mtth.svg)](https://saucelabs.com/u/buffer)

Various distributions are available when using browserify, depending on which
module you `require`:

+ `avsc/etc/browser/avsc.js`: the full distribution (~49kB minified and
  compressed), including serialization, protocols, and Avro file support. This
  is also the default distribution you get when you `require('avsc')` directly.
+ `avsc/etc/browser/avsc-prototols.js`: a slightly lighter distribution (~32kB)
  which doesn't include file support.
+ `avsc/etc/browser/avsc-types.js`: the lightest distribution (~20kB) which
  only includes serialization support.
