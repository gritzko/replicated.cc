---
layout: default
title: Nominal format
in_section: specs
---

# Nominal format

Nominal RON is a simple memory layout that stores uncomressed RON ops. It serves three purposes:

- exchange RON ops within the same address space,
- define the canonical representation for e.g. content hashing,
- serve as an intermediary format in any conversions, mappings, filters and transformations.

Normally, a serialized RON frame is read by an iterator/parser/cursor which creates a nominal-RON representation of each next op. 
A builder/serializer/writer converts a nominal-RON op into the resulting format.

The nominal format per se is not intended as a frame serialization format, as it would be too inefficient.

The nominal RON may help greatly in mixed environments, e.g. when a C++ implementation is used from a node.js program through the bindings.
If the js part may consume/produce nominal-RON data, it needs no own parsers, no own builders.
Instead, it may consume parsed data from a scratchpad buffer.
The same applies to bizzare situations, e.g. C++ engine/parsers/builders running in a browser under wasm, used by a Java program running under GWT.

## Overall layout

The nominal format is made of 128-bit atoms, two 64-bit words each.
Words use the 4+60 bit layout.
Nominally, the byte layout is big-endian (e.g. a hash function expects big-endian words).
Although, an implementation may use little-endian words if requested.
All pictures assume big-endian words.

In most cases, the origin word specifies a byte range in the original RON buffer (text RON, binary, JSON, CBOR).
The maxumum allowed frame size is 1GB, so the byte range is given as two 30-bit unsigned ints for the offset and the length.
The raw buffer is supposed to be available.


## Frame header

(rarely used)

Bytes 0-3 (`mmmm`) contain magic bytes, `RON2` for closed RON, `ROP2` for open.

Bytes 4-7 (`cccc`) is the number of ops in the frame, as a big-endian integer.

Bits 4-7 in byte 8 are set to `1100`.

Bytes 8-15 contain the byte range within the raw unparsed buffer.

```
mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm cccccccc cccccccc cccccccc cccccccc
1100oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll
```

Again, frame header is supposed to be rarely used, cause entire frames are rarely uncompressed at once.

## Op header

(less rarely used)

Bytes 0-3 (`mmmm`) contain magic bytes, `ron2` for closed RON, `rop2` for open.

Bytes 4-7 (`cccc`) contain the number of atoms in the op, as a big-endian integer.

Bits 4-7 in byte 8 (`11__`) indicate the op type:

- `1100` raw,
- `1101` reduced,
- `1110` header,
- `1111` query.

Bytes 8-15 contain location of *raw buffer* location with unparsed op content.

```
mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm cccccccc cccccccc cccccccc cccccccc
11__oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll
```

## UUIDs

A 128-bit RON UUID:

```
vvvv···· ········ ········ ········ ········ ········ ········ ·······
00ss···· ········ ········ ········ ········ ········ ········ ·······
```

Bits 4-7 in byte 0 (`vvvv`) represent UUID *variety.*

Bits 6-7 in byte 8 are set to `00`, indicating that this atom is a UUID.

Bits 4-5 in byte 8 (`ss`) represent UUID *version*.

(That is the usual RON UUID in-memory layout, except implementations are likely to use platform endiannes internally.)


## Integers

Bytes 0-7 (`iiii`) a 4-bit signed integer.

Bytes 8-15 contain location of *raw buffer* location with unparsed integer representation.

Bits 4-7 in byte 8 are set to `0101`, indicating that this atom is integer atom.

60 value bits of the origin word (`ooo`, `lll`) contain the raw buffer range for the original unparsed value.
`ooo` is the offset, `lll` is the length, both are 30-bit unsigned integers.

```
iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii
0101oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll
```

## Floats

Bytes 0-7 (`ffff`) contain a 64-bit float (should be IEEE 754).

Bits 4-7 in byte 8 are set to `0111`, indicating that this atom is a float.

Bytes 8-15 contain a *raw buffer* range for the unparsed float representation, same as integer.

```
ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff
0111oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll
```

## Strings

The value word contains string metrics: byte length `bbb` and codepoint length `ccc`.
These metrics are relative to a pure UTF-8 string, all escapes etc replaced.

Bits 4-7 in byte 8 are set to `0110`, indicating that this atom is a string.

Bytes 8-15 contain a *raw buffer* range (note the raw byte length may not match the UTF-8 byte length).

```
0000bbbb bbbbbbbb bbbbbbbb bbbbbbbb bbcccccc cccccccc cccccccc cccccccc
0110oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll
```

## Read next

[Binary format](../binary/).
