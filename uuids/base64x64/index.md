---
layout: default
title: Base 64×64
in_section: uuids
---

# Base 64×64

RON uses a custom base64 serialization to encode arbitrary sequences of 60 bits:

- Base64 characters go in their ASCII order: `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_abcdefghijklmnopqrstuvwxyz~`,
- only 60 least significant bits are encoded (10 chars)
- Tailing zeroes are skipped, e.g. `0230000000` is shortened to `023` (this preserves the alphanumeric order).

The tail-skip rule is a bit counter-intuitive, as the traditional Arabic notation skips leading zeroes (`0000123` equals `123`).
Unlike Arabic, in Base 64×64 lexigraphical order of encoded strings matches numerical order of corresponding numbers.

Some examples:

- `80` = `"000000001G"`
- `463490553` = `"00000Rd4su"`
- `576460752303423488` = `0x800000000000000` = `"W000000000"` = `"W"`

A bit counter-intuitively, all predefined string constants (like operation names, up to 10 chars) can be decoded as very big numbers.
For example, `on` decodes to 932808072819113984 (`on` = `"on00000000"` = `0xCF2000000000000`).

If compared to the RFC4122 coding, RON UUIDs are:

* shorter,
* somewhat more human readable and
* maintain meaningful order in binary and text alike.

## Read next

Replica [assignments rules](../replicas/).
