---
layout: default
title: RDTs
section: rdts
---

# Replicated Data Types

RDT are Swarm term for object types. Each object has an associated RDT specified at creation time. Swarm uses that RDT to correctly process individual operations and reduce them in a consistent object state.

Internally RDT works as pure functions: `f(state_frame, change_frame) → new_state_frame`. Here frames are either empty frames or single ops or products of past reductions by the same RDT.

RDTs are:

- associative, e.g. `f(f(state, op1), op2) == f(state, patch)` where `patch == f(op1,op2)`,
- commutative for concurrent ops (can tolerate causally consistent partial orders), e.g. `f(f(state,a),b) == f(f(state,b),a)`, assuming `a` and `b` originated concurrently at different replicas,
- idempotent, e.g. `f(state, op1) == f(f(state, op1), op1) == f(state, f(op1, op1))`,
- optionally, RDTs may have stronger guarantees, e.g. full commutativity (tolerates causality violations).

A `change_frame` could be an op, a patch or a complete state. Hence, a baseline RDT can “switch gears” from pure op-based CRDT mode to state-based CRDT to delta-based:

- `f(state, op)` is op-based,
- `f(state1, state2)` is state-based,
- `f(state, patch)` is delta-based.

## Mappers

A mappers are Swarm term for “views” in a database. A mapper translates a replicated object's state in RON format into other formats:

- Mappers turn Swarm objects into JSON or XML documents, C++, JavaScript or other objects.
- Mappers are two-way interfaces to raw RDTs. You can read object’s view through mappers and “write” into mappers. Mapper will convert all operations internally into RDT operations.
- Mappers can be pipelined, e.g. one can build a full RON → JSON → HTML [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) app using just mappers.

## Built-in RDTs

- [LWW: Last-Write-Wins Object](lww/)
- [RGA: Replicated Growable Array](rga/)
- [TXT: Collaborative text](txt/)
- [Sets](set/)
- [Counters](counter/)
- [Binary blobs](blob/)

## See also

[RON-RDT Composition](composition/).