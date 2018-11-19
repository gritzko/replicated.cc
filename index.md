---
layout: default
title: Replicated Object Notation
---

# Replicated Object Notation

Replicated Object Notation (RON) is a format for *distributed live data*. RON is suitable for optimized storage, efficient transmission, reliable negotiation and queries. RON was concieved as a network-native and efficient alternative to JSON and other serialization formats.

<img class="fig" src="i/cover.jpg">

## Principles

Key RON principles are:

- **Immutability**. Once created RON operations travel through system unchanged.
- **Addressability**. Everything: changes, versions, every piece of data is uniquely identified and globally referenceable.
- **Source and transport agnosticism**. It doesn’t matter where RON data comes from and how. Any node can read RON from the network, filesystem, database, message bus and/or local cache, in any order, and merge them correctly.
- **Efficiency**. RON data is optimized to make metadata overhead tolerable and real-time collaborative applications possible. It’s optimized machine first, human readability second.
- **Causality**. Each RON operation explicitly references what version of an object it is based on. No matter how and when you get your data, you can always reconstruct the correct order events have happened in.

RON is an answer to the new reality where different agents join and leave network all the time and communicate with each other without any order or reliability guarantees. In other words, RON is lingua franca for Generation Internet.

## Quick look

This RON frame creates two LWW objects and sets two fields in each:

```
@1D4ICCA+XU5eRJ :lww!
@1D4ICCB+XU5eRJ :1D4ICCA+XU5eRJ 'x' 1,
@1D4ICCC+XU5eRJ :1D4ICCB+XU5eRJ 'y' 'A',
@1D4IDD0+XU5eRJ :lww!
@1D4IDD1+XU5eRJ :1D4IDD0+XU5eRJ 'a' 3.14159,
@1D4IDD2+XU5eRJ :1D4IDD0+XU5eRJ 'b' 'Y',
.
```

For explanation, see [UUIDs](/uuids/) and [specification](/specs/).

