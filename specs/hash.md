# Merkle hashing

Merkle hashing guarantees data integrity.
It is used extensively in numerous high-profile project: git, BitTorrent, BitCoin and other cryptocurrencies.
With RON, the challenge is efficiency.
RON ops are very fine grained pieces of data; likely, those are smaller than their hashes.
Hence, hash overhead becomes a part of the bigger issue of metadata overhead.

Plus, RON supports various serializations; a hash of a text frame would be different from the hash of a CBOR frame.
Even within one serialization, there are various format options, such as whitespace and compression.
For that reason, the hash is defined on the [nominal format](nominal/).
It mostly uses uncompressed 128-bit atoms, hence (almost) no optional elements.
That roughly matches in-memory layout of a parsed op in compiled languages (e.g. C++, go, also long-based Java).

This spec defines the op data which has to be fed to a hash function to produce op hashes, recursively.
We assume nominal big-endian 64-bit words, which may differ from de-facto representation (little-endian words).

Key concerns of the algorithm are:
* Build a proper Merkle structure along the lines of the Lamport process/message graph.
* Ensure that different ops don't match to the same byte sequence and hence hash (mocking).
* Ensure any serialization of an op maps to the same hash.

## Specifier (op key)

The op's id and reference form the Merkle DAG structure of the op graph.
The hashing function is fed two pairs UUID+hash, as produced for earlier ops by the same function:
* a pair of the op's own id (128-bit UUID) and a hash of the immediate preceding op by the same origin.
* a pair of the reference UUID and a hash of the referenced op.
This part has a size of `(16+HASHSIZE)*2` bytes, e.g. 96 bytes for any 32-byte hash function.

## Value

Value atoms are fed in the nominal representation, 128 bit per atom, values parsed, origin ranges cleared, all big-endian.
Effectively, the second word (origin) only contains the flags.
Supplying the flags is necessary to prevent mocking, e.g. picking an integer that matches some float in its bit layout, etc.
Effectively, UUIDs are fed as-is (two big-endian words), tombstone flag cleared.
Value UUIDs don't form Merkle links, unless explicitly accompanied by hashes.

The special case is strings, because:

1. They are arbitrary-length.
2. Unicode string serialization has some degrees of freedom, esp. the text representation (e.g. \uXXXX escapes).
3. Mocking.

Strings are fed as 128-bit atoms followed by arbitrary-length UTF-8 strings.
The atom has codepoint length and byte length values set, origin ranges cleared.
That is followed by the string itself, as a correct UTF-8 sequence of the specified byte/codepoint length.

