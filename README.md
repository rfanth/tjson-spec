# Text JSON (TJSON) - a readability optimized, round trip compatible alternative to JSON

## Why Text JSON (TJSON)?

Humans read from screens in squares.  Computers read character by character, ignoring linefeeds, making JSON look just fine, but any attempt to turn JSON into normal text generally looks like a thin line passing through the screen going down to infinity.  Pretty printing helps, but it can't address many of the core readability problems.  This is my attempt to fix that.

## Specification

The full specification is in [tjson-specification.md](tjson-specification.md).

## Reference Implementation

[TJSON reference implementation written in Rust](https://github.com/rfanth/tjson/)
The TJSON reference implementation includes a full featured binary and library.  The library compiles with musl, and has WASM compatibility for use in webpages.

[TJSON test suite](https://github.com/rfanth/tjson-tests)

## Home Page and Demo

[textjson.com](https://textjson.com)

## Author

Original TJSON concept and specification created by R.F. Anthracite (rfa@rfanth.com)
