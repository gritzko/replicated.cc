---
layout: default
title: RON Specification
section: specs
---

## RON Serialization

There are four different ways to represent RON structures:

1. *Textual* is good for human-friendly output, commands, queries and debugging. Specifications for:
  - [Atoms: integers, floats, strings](atoms/);
  - [Operations](ops/);
  - [Frames](frames/).
2. [*Nominal*](nominal/) is for internal in-memory representation.
3. [*Binary*](binary/) is for machine-to-machine communication and persistence.
4. [*JSON/CBOR*](json/) is for JSON-centric systems.

## RON crypto

[Merkle hashing](hash/).

## RON Communication protocol

[Transactional/atomic changes](changes/).

[Network protocol](network/).
