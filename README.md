cbor-test-vectors
============

Reporitory of encoded CBOR values in machine-processable form for parser testing. See [vectors.json](vectors.json).

The basis of the repository are the examples from [Appendix A of RFC 8949](https://www.rfc-editor.org/rfc/rfc8949.html#name-examples-of-encoded-cbor-da).
All examples in Appendix A of RFC 7049, encoded as a JSON array.

Each element of the test vector is a test case map (JSON object) with the keys:
- `hex`: HEX encoded CBOR data
- `flags`: Set of these flags (some are mutually exclusive):
  - `valid`: This is valid and should be parseable by any parser that has all required features
  - `invalid`: This is invalid and should not be parsed under any circumstances
  - `canonical`: This value, as it is, is canonically encoded ([description of what is canonical](https://www.rfc-editor.org/rfc/rfc8949.html#name-deterministically-encoded-c))
  - `float`: The diagnostic contains floating point numbers. Since formatting floats greatly differs from platform to platform and is not always easily configurable, you may want to special-case failures on these tests.
- `features`: Each parser is built with different purpose in mind and may handle corner-cases differently. This is a set of features, which are present in this CBOR value. If your parser does not support all features of a test case, it may be OK to skip that test.
  - When the tag begins prefixed with `!` it means that the test case is relevant only when the feature is NOT implemented by the parser.
  - `int63`: Major type 0 and 1 with precision up to -2^63..2^63-1 (integers representable by signed int64, e.g. Java's `long`)
  - `int64`: Major type 0 and 1 with full precision, -2^64..2^64-1 (typically requires special care by the implementation and may not be as efficient)
  - `mapdupes`: Duplicate keys in maps
  - `simple`: Support for simple values (major 7) other than `true`, `false`, `null` and `undefined`
  - `float16`: Support for reading and writing float16 format (in major 7)
  - `bignum`: Support for parsing tags 2 and 3 as numbers of arbitrary precision
- `diagnostic`: The representation of the data item in CBOR diagnostic notation, without encoding marks.
  - Strings use JSON syntax with only mandatory escapes (see [ECMA-404](https://www.ecma-international.org/wp-content/uploads/ECMA-404_2nd_edition_december_2017.pdf)). The file itself is encoded using UTF-8.
- `diagnosticExact`: Same as `diagnostic`, but contains all encoding markers.
- `canonical`: When the test case is not canonical (`flags` does not contain `canonical`), encoding it in canonical form will yield this CBOR HEX encoding

### Expected features
These features are expected to always work (are not flagged with a feature flag):
- Major type 0 and 1 (unsigned and negative integer) with precision up to ±2^53 (integers representable by float64, e.g. JavaScript's number)
- Major type 2 (byte string) with definite and indefinite length
- Major type 3 (UTF-8 text) with definite and indefinite length, checking for UTF-8 validity is not required
- Major type 4 (array) with definite and indefinite length
- Major type 5 (map) with definite and indefinite length with support of any key-value types, support for duplicate keys is not required
- Major type 6 (tag) without any explicit support for tag semantics (just preserving the tag is enough)
- Major type 7 (simple & float) float32 and float64 is required, float16 is optional. Support for simple values outlined in main RFC (true, false, null, undefined), support for other simple values (that are without RFC assigned meaning) is not required.

Given the nature of the test value repository, size limits are unlikely to be a problem.
