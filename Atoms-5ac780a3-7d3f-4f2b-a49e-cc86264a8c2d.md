# Atoms

Strings are serialized as UTF-8.

Integers are serialized using the zig-zag coding (the l.s. bit conveys the sign).

Floats are serialized as IEEE 754 floats (4-byte and 8-byte support is required, other lengths are optional).

##