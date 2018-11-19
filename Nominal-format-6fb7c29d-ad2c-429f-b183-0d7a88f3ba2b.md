# Nominal format

Nominal RON is a simple memory layout that stores uncomressed RON ops. It serves three purposes:

- exchange RON ops within the same address space,
- define the canonical representation for e.g. contnet hashing,
- serve as an intermediary format in any conversions, mappings, filters and transformations.

Normally, a serialized RON frame is read by an iterator/parser/cursor which creates a nominal-RON representation of each next op. A builder/serializer/writer converts a nominal-RON op into the resulting format.

Nominal format per se is not for serialization, it would be too inefficient to serialize.

The nominal RON may help greatly in complicated situations, e.g. when a C++ implementation is used from a node.js program through the bindings. If the js part may consume/produce nominal-RON data, it needs no own parsers, no own builders. Instead, it would consume parsed data from the same scratchpad buffer. The same applies to extremely bizzare situations, e.g. C++ engine/parsers/builders running in a browser under wasm, used by a Java program running under GWT.

# Overall layout

The nominal format uses fixed-width 128-bit cells with 4+60 bit layout. Byte layout is big-endian.

    <frame cell> [<op cell> <object uuid> <ref uuid> [<atom>]*]+

# Frames

Bytes 0-3 (`mmmm`) contain magic bytes, `RON2` for closed RON, `ROP2` for open.

Bytes 4-7 (`cccc`) contain amount of ops in frame as big-endian integer.

Bits 4-7 in byte 8 are set to `1111`, indicating that this cell contains frame.

Bytes 8-15 contain location of *raw buffer* location with unparsed frame content.

    mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm cccccccc cccccccc cccccccc cccccccc
    1111oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll

Frames format is supposed to be rarely used, cause entire frames are rarely uncompressed at once.

# Ops

Bytes 0-3 (`mmmm`) contain magic bytes, `ron2` for closed RON, `rop2` for open.

Bytes 4-7 (`cccc`) contain amount of atoms in op as big-endian integer.

Bits 4-7 in byte 8 (`10__`) indicate op type:

`1000` raw,

`1001` reduced,

`1010` header,

`1011` query.

Bytes 8-15 contain location of *raw buffer* location with unparsed op content.

    mmmmmmmm mmmmmmmm mmmmmmmm mmmmmmmm cccccccc cccccccc cccccccc cccccccc
    10__oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll

# UUIDs

All 128 bit store RON UUID:

    vvvv···· ········ ········ ········ ········ ········ ········ ·······
    00ss···· ········ ········ ········ ········ ········ ········ ·······

Bits 4-7 in byte 0 (`vvvv`) represent UUID *variety.*

Bits 6-7 in byte 8 are set to `00`, indicating that this cell contains UUID.

Bits 4-5 in byte 8 (`ss`) represent UUID *version*.

# Integers

Bytes 0-7 (`iiii`) contain big-endian integer.

Bytes 8-15 contain location of *raw buffer* location with unparsed integer representation.

Bits 4-7 in byte 8 are set to `0101`, indicating that this cell contains integer.

Next 30 bits in bytes 8-12 (`oooo`) contain raw buffer offset as big-endian integer.

Last 30 bits in bytes 12-15 (`llll`) contain raw buffer length as big-endian integer.

    iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii iiiiiiii
    0101oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll

# Floats

Bytes 0-7 (`ffff`) contain 64-bit float (should be IEEE 754).

Bits 4-7 in byte 8 are set to `0111`, indicating that this cell contains float.

Bytes 8-15 contain location of *raw buffer* location with unparsed float representation, same as integer.

    ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff
    0111oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll

# Strings

Bytes 0-7 (`sss000`) contain up to 8 first characters of string, padded from the right with zeroes.

Bits 4-7 in byte 8 are set to `0110`, indicating that this cell contains string.

Bytes 8-15 contain location of *raw buffer* location with unparsed string content.

    ssssssss ssssssss ssssssss ssssssss ssssssss ssssssss ssss0000 00000000
    0110oooo oooooooo oooooooo oooooooo oollllll llllllll llllllll llllllll