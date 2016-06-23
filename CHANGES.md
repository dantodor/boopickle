# Changes

## 1.2.3

Again some performance tuning. BooPickle now allows more control over choosing speed over size.

- Performance fix for really large datasets where the `IdentMap` implementation fails totally as the number of references grows over 10k
- Deduplication of references and immutable objects is now selectable via `PickleState` and `UnpickleState` constructors
- `String` pickler is again doing deduplication (if deduplication of immutable objects is enabled)
- A bug in `IdentMap` implementation caused errors when two different objects had same identity hash value

## 1.2.2

- Quick bug fix for UTF8 encoding on browsers with `TextEncoder` support

## 1.2.1

This is a speed optimization release focusing on reducing encoding overhead especially when pickling small objects. These changes should not affect
user code, but the format of encoding has changed, so something pickled with a previous version will not unpickle properly on this version and vice versa.

- Several optimizations to reduce overhead of pickling small data.
  - removed immutable references from pickle states
  - deduplication is now optional (enabled by default, disable using a constructor parameter on `PickleState` and `UnpickleState`)
  - strings are no longer deduplicated to avoid a performance hit
  - collections are no longer deduplicated
  - custom fast identity map class for managing identity references while pickling
  - smaller initial encode buffer
- `CompositePickler` performance improved when pickling a composite with a lot of subtypes
  - the "copy constructor" is removed from `CompositePickler` as a result of this change. Use `.join` instead

### Other changes
- Array size is always written as a 32-bit integer to ensure proper alignment (big boost when reading/writing floats/ints in JS) and is padded with
  extra 32-bits to ensure proper alignment for Double arrays
- Separate buffer pools for heap and direct `ByteBuffer`s and an optimized implementation of the pool
- Most picklers now support encoding a `null` reference
- Removed `reset()` from `BufferProvider`. Create a new instance instead, it's now cheap :)

## 1.2.0

- Extracted common `Encoder` and `Decoder` traits with separate implementations for size and speed. Default codec is optimized for size
  - Added a codec optimized for speed, using simpler encoding. This is intended to be used within an application, for example when communicating between
    web workers in Scala.js
  - `Unpickle.fromBytes` now takes an implicit for building an `UnpickleState` for a given `ByteBuffer` to allow selection between different decoders
- Removed special encodings for UUID and numeric strings
- Special codecs for common `Array` types (`Byte`, `Int`, `Float`, `Double`) and optimized code path for pickling them
- Infrequently used picklers made `lazy` to improve Dead Code Elimination (DCE) in Scala.js
- String coding performance improved on JVM
- Updated to Scala.js 0.6.9
- `transformPickler` parameter order switched to better support type inference
- Introduced a `BufferPool` for reusing `ByteBuffer`s when pickling. You can return used buffers back to the pool with `BufferPool.release`

## 1.1.3

- Support `null` `UUID`s. `UUID`s still take 16 bytes, except for `null` and `00000000-0000-0000-0000-000000000000` which take 17.

## 1.1.2

- Added support for `sealed abstract class` hierarchies (fix #37)
- Updated to Scala.js 0.6.6 (due to [#2158](https://github.com/scala-js/scala-js/issues/2158))
- Updated other pickling libs to latest version in performance tests

## 1.1.1

- Picklers for `BigInt` and `BigDecimal` (by @guersam)
- Performance tests for Circe 0.2.0

## 1.1.0

This version has several backward-compatibility breaking changes. Most notably you should change your `import boopickle._` into
`import boopickle.Default._`, which should be enough for most common cases. If you have written your own picklers, you must merge
the unpickling functionality into the pickler. There are also changes to how `CompositePickler`, `TransformPickler` and
`ExceptionPickler` are used.

- Moved all implicits into `boopickle.Default` to better control what implicits are imported
- Unpicklers merged into Picklers, so there are no separate Unpicklers anymore
- Added helper functions `compositePickler`, `transformPickler` and `exceptionPickler` in `Default`
- BooPickle generated macros are now compatible with *-Xstrict-inference* compiler option
- Trait hierarchies with type parameters can now be pickled automatically (by @FlorianKirmaier)
- Improved error messages
- Performance tests now use uPickle 0.3.4

## 1.0.0

- Support for auto-generation of `CompositePickler` for sealed trait class hierarchies
- When a `ByteBuffer` is pickled, it now retains its byte order when unpickled
- Refactored String coding in Scala.js

## 0.1.4

- Fixed a bug in decoding strings from a `ByteBuffer` with an array offset
- Added transformation picklers to help creating custom picklers
- Added special support for pickling Exceptions

## 0.1.3

- Fixed a bug in byte order when unpickling a `ByteBuffer`
- Enforce byte ordering before unpickling
- `CompositePickler` supports `join` method to pickle deeper type hierarchies
- Use heap `ByteBuffers` on JVM by default, direct on JS for optimal performance

## 0.1.2

- Support for heap and direct byte buffers (and custom ones, too)
- Support for returning a sequence of ByteBuffers instead of a combined one
- Changed to little endian, and updated integer encoding scheme for negative numbers
- Fixed a bug in unpickling a ByteBuffer
- Optimized string decoding in case of heap buffer

## 0.1.1

- Functions in Un/PickleState were private, so macros did not work outside the boopickle package!
- TextEncoder produces Uint8Array which needs to be cast to Int8Array for ByteBuffer to work
- Added pickler for ByteBuffer (mainly to make BooPickle work easily with Autowire)

## 0.1.0

- Initial release