---
layout: default
title: Atoms
in_section: specs
---

# Atoms: integers, floats, strings

For object content, RON defines four basic data types.

- *Integers* start with equal sign `=1099`.
- *Floats* start with `^`: `^3.14159`, `^2.9979E5`.
- *Strings* are quoted with single quote: `'Hi'`. String support backslash-escaped `\'` `\\` `\n`.
- *UUIDs* start with `>` sign: `>A/D4ICCE+XU5eRJ`.

When unambiguous, prefixes could be omitted: `1099` `3.14` `D4ICC+XU5eRJ`.

If UUID happen to serialize to a string that could also be interpreted as a number (e.g. `0/1230000000$0000000000` = `123`), it MUST be prefixed: `>123`.

## Read next

[Operations](../ops).
