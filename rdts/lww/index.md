---
layout: default
title: LWW
in_section: rdts
---

# LWW: Last-Write-Wins Object

- Key-value interface, string keys, arbitrary type values.
- Concurrent operations are applied in chronological order, overriding every other operations that have happened before.

Create an LWW object:

```
@123+origin :lww ; .
```

Set field `'age'` to value `18`:

```
@124+origin :123+origin 'age' 18 ; .
```

Delete a field:

```
@125+origin :124+origin 'age' ; .
```

## Read next

[RGA: Replicated Growable Array](../rga/).