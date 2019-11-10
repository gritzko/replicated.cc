# RONv -- binary, varint based

RONv encodes a RON frame as a stream of varints (LEB-128, aka protobuf varint).

## Unboxed coding

Unboxed coding is applicable to single atoms, if their type is known in advance:

* INT is encoded as a zig-zag coded varint (1..10 bytes, the sign in the lowest bit)
* FLOAT is taken as 64-bit IEEE 754, *flipped* and coded as a 64-bit int (see below on flipping)
* ID is encoded depending on the version:
    * NAME, SCOPED and LAMPORT are coded as two 64 bit varints: first origin, then value (both are *flipped*)
    * YOLO is coded as a 64-bit origin (flipped) then two 32-bit varints for yndx and hndx (unflipped).
* STRING is always boxed

Flipping is applied to 64-bit words which likely have their most-significant bits set, while least-significant bits are often unset (zeros).
That is the opposite of what varint coding expects (small ints have l.s. bits set).
Hence, we byte-flip 64-bit words, like in the endianness conversion.
The 7th byte (m.s.) becomes 0th byte (l.s.), 1st becomes 6th, and so on.
The resulting varint likely takes less space.

## Boxed coding

In the boxed coding, an atom is coded as a "box descriptor" varint followed by the unboxed atom value.
A box descriptor varint has the atom type (INT, ID, STRING, FLOAT) in its l.s. two bits.
The rest of the bits tell the byte length of the unboxed value.
0 length is valid, corresponds to the default value (0, NIL, "", 0.0 respectively).

STRING is coded as a box descriptor followed by varints for its Unicode code points.
For ASCII symbols that is exactly the same value.
For multi-byte symbols, LEB128 is slightly more efficient (e.g. for Japanese texts).

One may ask, why don't we use UTF-8 for strings?
Well, UTF-8 is itself a varint format.
The reason to use LEB128 is simple: depending on the exact output format, strings have to be recoded anyway (escaped, unescaped, converted).
For example, CSV, RONt and JSON all need different escapings of UTF-8.
A Java or JavaScript program may need UTF-16 internally.
While the literal UTF-8 can be memcpy'd around in many cases, that opportunity is more relevant for the caching layer.
Internally, RONv uses varints for everything.

Hence, number 0 if coded as `00` (hex). Number 1 is coded as `04 01`.
An empty string is `02`. String "abc" is `0e 61 62 63`.

## Pallets

A "pallet" is a packing of boxes.
Pallets can be uniform (all atoms same type) or non-uniform.
A pallet consists of a pallet descriptor (a varint) and a sequence of either boxed (non-uniform) or unboxed (uniform) atoms.
The pallet descriptor is very much like a box descriptor: a 32 bit varint containing an atom type in its l.s. two bits and the payload byte length in the rest of the bits.
The atom type 3 (STRING) corresponds to a non-uniform pallet where all atoms are boxed individually.

Empty pallet: `00`.
A pallet containing string "abc": `13 0e 61 62 63`.
A pallet containing numbers 1, 2, 3: `0c 01 02 03`.
A pallet containing string "abc" and numbers 1, 2, 3: `2b 0e 61 62 63 04 01 04 02 04 03`.

## Ops in RONv

...