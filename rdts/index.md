---
layout: default
title: RDTs
section: rdts
---

# Replicated Data Types

In RON, an RDT is the data type that an object inherits from. Each object has an associated RDT specified at creation time (e.g. the root op might be `:lww` or `:rga`). RON uses the rules associated with that RDT to correctly process individual operations and reduce them to a consistent object state.

Internally, RDTs works as pure functions: `f(state_frame, change_frame) → new_state_frame`. Where frames are either empty frames or single ops, or products of past reductions by the same RDT. A `change_frame` could be an op, a patch or a complete state.

All RDTs are:

- associative, e.g. `f(f(state_frame, op1), op2) == f(state_frame, patch)` where `patch == f(op1,op2)`,
- commutative for concurrent ops (can tolerate causally consistent partial orders), e.g. `f(f(state_frame, a), b) == f(f(state_frame, b), a)`, assuming `a` and `b` originated concurrently at different replicas,
- idempotent, e.g. `f(state_frame, op1) == f(f(state_frame, op1), op1) == f(state, f(op1, op1))`,
- optionally, RDTs may have stronger guarantees, e.g. full commutativity (tolerates causality violations).

One of the central features of RDTs is the ability to “switch gears” from pure op-based CRDT mode, to patch-based CRDT, to state-based:

<table>
  <thead>
  <tr>
    <th>"Gear"</th>
    <th>Function Application</th>
    <th>When Useful</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>Op-based</td>
    <td>`f(state, op)`</td>
    <td>Real-time Sync</td>
  </tr>
  <tr>
    <td>Patch-based</td>
    <td>`f(state, patch)`</td>
    <td>Periodic Sync</td>
  </tr>
  <tr>
    <td>State-based</td>
    <td>`f(state1, state2)`</td>
    <td>Full Reconciliation</td>
  </tr>
  </tbody>
</table>

## Mappers

A mapper is a Swarm term akin to a “view” in a database. A mapper translates a replicated object's state in RON format into other formats:

- Mappers turn Swarm objects into JSON or XML documents, C++, JavaScript or other objects.
- Mappers are two-way interfaces to raw RDTs. You can read a view of an object from its mapper and “write” into its mapper. A mapper will convert all operations internally into RDT operations.
- Mappers can be pipelined, e.g. one can build a full RON → JSON → HTML [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) app using just mappers.

## Built-in RDTs

- [LWW: Last-Write-Wins Object](lww/)
- [RGA/CT: Replicated Growable Array / Causal Tree](rga/)
- [TXT: Collaborative text](txt/)
- [Sets](set/)
- [Counters](counter/)
- [Binary blobs](blob/)

## See also

[RON-RDT Composition](composition/).