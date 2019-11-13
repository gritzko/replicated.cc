---
layout: default
title: RON Specification
section: specs
---

## RON Serialization

RON has to ensure bitwise convergence of replicas - both as a guarantee of correctness and as a prerequisite for using any crypto.
It helps that RON ops are immutable, but specifying a bit-precise protocol is a surprisingly difficult topic.
UUIDs and integers are easy to specify, apart from endianness issues.
The main headaches come from strings and floating-point numbers.
Strings come in various encodings; Unicode, for example, is notoriously complex.
Floating-point numbers are not deterministic under some operations, e.g. when added or converted between representations.
Also, every format has its bitwise liberties; binary may have compression, text has whitespace, etc.

Hence, we define the *nominal* RON format based on binary fixed-width big-endian UTF-8 bit layout.
Hashes and algorithms are defined relative to the nominal format.
The nominal format is similar to the in-memory representation used by compiled languages.
That said, the literal nominal serialization may not be used on network or disk at all, except transiently in the hashing algorithm.

Furthermore, textual RON, binary RON and others are merely "serializations" of the nominal RON.
Serializations may take any number of liberties--compression, whitespace, etc.--as long as there is a two-way mapping between a serialization and the nominal format (a bijection).

At present, there are four different ways to represent RON structures:

0. [The *Nominal* RON](nominal) as the basis.
1. The *Textual* RON for human-friendly output, commands, queries and debugging. See specifications for:
  - [Atoms: integers, floats, strings](atoms);
  - [Operations (ops)](ops);
  - [Entire RON frames](frames).
3. [The *Binary* RON](binary) is for machine-to-machine communication and persistence.
4. [*JSON/CBOR*](json) is for JSON-centric systems.


## RON crypto

[Merkle hashing](hash).

## RON Communication protocol

[Transactional/atomic changes](changes).

[Network protocol](network).
