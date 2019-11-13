---
layout: default
title: Atoms
in_section: specs
---

# Binary format

The binary RON format is more efficient than either the nominal or text formats because of higher bit density. It is also simpler and safer to parse because of explicit field lengths. Obviously, it is not human-readable.

Like the text format, the binary format is only optimized for iteration. Because of compression, records are inevitably of variable length, so random access is not possible. Also, compression depends on iteration, as UUIDs get abbreviated relative to similar preceding UUIDs.

This document will use the standard "m.s." and "l.s." abbreviations to refer to "most significant" and "least significant" bits.

A binary RON frame starts with the magic bytes `RON2`, followed by the frame length. The rest of the frame is a sequence of *fields*. Each field starts with a *descriptor* specifying the type of the field and its length.

The frame length is serialized as a 32-bit [big-endian](https://en.wikipedia.org/wiki/Endianness) integer. The maximum length of a frame is 2<sup>30</sup> bytes (a [gibibyte](https://en.wikipedia.org/wiki/Gibibyte)). If the length value has its m.s. bit set to 1, then the frame is *chunked*. A chunked frame is followed by a continuation frame. A continuation frame has no magic bytes, just a 4-byte length field. The last continuation frame must have the m.s. bit of its length set to 0.

A descriptor's first byte uses the four m.s. bits to describe the type of the field, while the other four bits describe its length.

```
7    6    5    4    3    2    1    0
+----+----+----+----+----+----+----+----+
| major   | minor   |     field         |
|    type |    type |        length     |
+----+----+----+----+----+----+----+----+
  128  64   32   16    8    4    2    1
   80  40   20   10    8    4    2    1
```

Field descriptor major/minor type bits are set as follows:

1. `00` RON op descriptor,
    - `0000` raw op,
    - `0001` reduced op,
    - `0010` header op,
    - `0011` query header op.
2. `01` Reserved (for binary data)
    - `0100` type (reducer) id,
    - `0101` object id,
    - `0110` event id,
    - `0111` ref/location id
3. `10` Atoms, compressed (zipped chains)
    - `1000` UUID, backreference
    - `1001` integer list
    - `1010` char list
    - `1011`
4. `11` Atom
    - `1100` UUID, uncompressed (lengths 1..16)
    - `1101` integer (big-endian, [zigzag-coded](https://developers.google.com/protocol-buffers/docs/encoding#signed-integers), lengths 1, 2, 4, 8)
    - `1110` string (UTF-8, length 0..2<sup>31</sup>−1)
    - `1111` float (IEEE 754-2008, binary 16, 32 or 64, lengths 2, 4, 8 resp)

A descriptor's four l.s. bits encode the length of the field in question. The length value given by a descriptor does not include the length of the descriptor itself.

If a field or a frame is 1 to 16 bytes long then it has its length coded directly in the four l.s. bits of the descriptor. Zero stands for the length of 16 because most field types are limited to that length. Op terms specify no length. With string atoms, zero denotes the presence of an extended length field which is either 1 or 4 bytes long. The maximum allowed string length is 1Gb (30 bits). In case the descriptor byte is exactly `1110 0000`, the m.s. bit of the next byte denotes the length of the extended length field (`0` for one, `1` for four bytes). The rest of the next byte (and possibly other three) is a big-endian integer denoting the byte length of the string.

Consider a time value query frame: `*now?.`

- 4 bytes are magic bytes (RON, `0101 0010 0100 1111 0100 1110 0011 0010`)
- frame length: 4 bytes (length 5, `0000 0000 0000 0000 0000 0000 0000 0101`)
- op term descriptor: 1 byte (`0011 0000`)
- uncompressed UUID descriptor: 1 byte (cited length 3, `0100 0011`)
- `now` RON UUID: 3 bytes (`0000 1100 1011 0011 1110 1100`, the "uncompressed" coding still trims a lot of zeroes, see below).

As UUID length is up to 16 bytes, UUID fields never use a separate length number. UUID descriptors are always 1 byte long. The length of 0 stands for 16.

Length bits `0000` stand for:

- zero length for op terms,
- 16 for integer/float atoms, zipped/unzipped UUIDs,
- for strings, that signals an extended length record (1 or 4 bytes).

An extended length record is used for strings because they can be up to 2GB long. An extended length record is either 1 or four bytes. Four-byte record is a big-endian 32-bit int having its m.s. bit set to 1. Thus, strings of 127 bytes and shorter may use a single byte length record.

## Ops

Op term fields may have a cited length of `0000`, or may be skipped if they match the previous op’s term. Still, sometimes we want to introduce redundancy, CRC/checksumming, hashing, etc. For precisely this reason we may use non-empty terms. The checksumming method is specified by the field length (TODO).

## Atoms

Strings are serialized as UTF-8.

Integers are serialized using [zig-zag coding](https://en.wikipedia.org/wiki/Variable-length_quantity#Zigzag_encoding) (the l.s. bit conveys the sign).

Floats are serialized as IEEE 754 floats (4-byte and 8-byte support is required, other lengths are optional).

## Read next

[JSON/CBOR format](../json).
