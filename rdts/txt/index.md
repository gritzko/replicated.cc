---
layout: default
title: TXT
in_section: rdts
---

# TXT: Collaborative text

`txt` is a plaintext mapper for `rga` reducer. It converts internal RGA structures to human-readable text.

First, you need to create RGA object (not txt, as it’s just a mapper):

```
@123+origin :rga ; .
```

Then subscribe to object’s updates through txt mapper:

```
@123+origin :txt ? .
```

Insert characters `'ABCDE'` at position `0`:

```
@124+origin :123+origin 0 'ABCDE';.
```

You should get an update from a server with what that text will look like:

```
@123+origin :txt !
@125+origin :123+origin 'A' ,
@126+origin :125+origin 'AB' ,
@127+origin :126+origin 'ABC' ,
@128+origin :127+origin 'ABCD' ,
@129+origin :128+origin 'ABCDE', .
```

Delete 3 characters starting from position 1:

```
@130+origin :129+origin 1 3;.
```

Note that all positions in insert/delete ops are version-relative. Position `1` means first character in a version of text that corresponds to object version `128+origin`.

Response to delete will be:

```
@123+origin :txt !
@131+origin :129+origin 'ACDE' ,
@132+origin :131+origin 'ADE' ,
@133+origin :132+origin 'AE' , .
```

## Read next

[Sets](../set/).