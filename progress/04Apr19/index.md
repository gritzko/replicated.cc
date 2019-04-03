# RFP 5 "RON" 6-month progress report

CRDT is a data replication/synchronization technology that needs no
central "master" replica. As such, it is often considered an essential
building block for distributed/decentralized systems.
While the core principles of CRDT are clear, there is certainly a gap
between theory and practice. Our research addresses the gap by
focusing on the following topics:

1. log-structured (aka operational) CRDTs; those can "switch gears"
between op-based, patch-based and state-based modes of operation
(those correspond to real-time sync, periodic sync and full
reconciliation).
2. minimizing metadata overhead by employing chain structures; the
effect is similar to the block-based approach, except the core model
stays simple.
3. a formal ACID 2.0 framework to mix different RDTs in the same
environment (ACID ~ associative, commutative, idempotent,
distributed).
4. finally, unifying Causal and Merkle structures (CRDTs and crypto).

That work is divided into three layers:

1. the foundation: an op-based formal data model and protocol
(Replicated Object Notation),
2. replicated data types (RDTs) based on the RON model,
3. on top of that, the (database) replication part.

For each of the layers, we provide three deliverables:

1. a formal specification,
2. algorithms with pragmatic complexity bounds, and
3. a reference implementation.

Overall, the aim is to make CRDTs a simple universal instrument within
the reach of every developer. Our work is online at
http://replicated.cc

## Progress report

```
+--------------+---------------+---------------+---------------+
|              |      RON      |      RDT      |    replica    |
+--------------+---------------+---------------+---------------+
|specification | 85% text based|50% lww,ct/rga | 50% branches  |
+--------------+---------------+---------------+---------------+
|algorithms    |      100%     |     75%       |     50%       |
+--------------+---------------+---------------+---------------+
|implementation|      50%      |     50%       |     25%       |
+--------------+---------------+---------------+---------------+
```

1. Replicated Object Notation: stable. The notation is subdivided into the
   [nominal format](http://replicated.cc/specs/nominal/) and various
   serialization formats. Algorithms and hashes are defined on the nominal
   format, which is zero-liberty, bit-precise. Serializations focus on
   practical issues: compression and performance (the binary format), human
   readability (text) or compatibility (JSON/CBOR).

    * RON-spec: the nominal format is finished, frozen. The text based
      version is mainly finished, short of some optimizations. The binary
      version needs work, JSON/CBOR versions are merely outlined.
    * RON-algo: finished, frozen (serialization-independent).
    * RON-code: API is done, the text-based variant of the protocol is done
      and frozen (fuzz-tested). The rest of the code is made in a way to
      make other serializations (binary, JSON/CBOR) pluggable.

2. Replicated Data Types: basic RDTs are finished, frozen.

    * RDT-spec: some RDTs had to be reworked to make them chain compression
      friendly. That is partly [documented](http://replicated.cc/rdts/rga).
      Although, a proper formal bit-precise specification is future work.
    * RDT-algo: finished parts are frozen. The key element here is the
      iterator-heap merge algorithm.
    * RDT-code: a variety of RDTs still need to be implemented. 

3. Replica: theory solved, implementation in progress.

    * Replica-spec: stable parts are the RocksDB-based key-value storage,
      connection state machine. The branch machinery is reasonably stable.
    * Replica-algo: patch production and data deduplication will need some
      work, the rest is reasonably stable.
    * Replica-code: [progressing](https://github.com/gritzko/ron-cxx/db/).
      The implementation is done in C++ for two reasons. First, we need
      deep integration with RocksDB. Second, we will need bindings to
      higher level languages later on.

Informally, we had to re-adjust primitives to address certain long-standing
problems (metadata size, merge algorithm performance).  That necessitated
readjusting all the other elements (e.g. the Causal Tree data struture, the
key-value storage architecture).

Similarly,  bitwise convergence and Merkle structures required a more
strict and formal approach to everything (the nominal format, Unicode
handling, endianness). That is plenty of tedious work.

The new exciting thing that is still evolving is *branches*.  It is an
approach that generalizes branches (in the distributed version control
sense) and dataset federation.  

## Future work

The goal of the first half of the project was to make our LEGO blocks; a
manageable coherent codebase we can build upon.  At this point, the
center of mass shifts to use-case testing and performance optimization.
(For example, branches fall in to the former category, binary serialization
falls into the latter.) 

A significant part of the future work is mappers.  Those are stateless
components converting between RON and other formats (plain text, CSV,
JSON). As mappers encapsulate CRDT/RON machinery, they greatly simplify
the creation of APIs and bindings for various languages and platforms
(priorities: golang, JavaScript/node.js and JavaScript/WebAssembly).



Finally, there is a lot of work creating/updating the documentation on
http://replicated.cc.


-----


# Protocol Labs RFP 5 proposal "Replicated Object Notation"

## Applicants

* Victor Grishchenko, victor.grishchenko@gmail.com
* Oleg Lebedev.

## Experience

* Victor Grishchenko, PhD
    * designed and implemented a swarming transport protocol
      (PPSP/libswift) which later became RFC 7574 (ed. A.Bakker et
      al), also the DAT protocol heavily relies on libswift/7574
      primitives
    * authored peer-reviewed papers on collaborative editors and CRDTs
    * designed and implemented a real-time editor for Yandex N.V.
      (top Russian Web search company)
    * worked for various companies as a real-time data sync expert
      (including Yandex, Realm, JetBrains)
* Oleg Lebedev
    * extensive experience in mobile and Web development
    * created the GraphQL API for the Swarm CRDT sync middleware
    * maintainer of the Swarm.js library

## Approach

Although the RFP is titled "Optimize storage and convergence time in
causal CmRDTs" we'd like to broaden the scope of it.  We believe, the
problem could not be solved within the bounds of CmRDT per se.  In the
recent years, there was certain convergence in the CRDT field, when
the border between CvRDT and CmRDT gradually blurred.  Eventually,
that resulted in a new breed of RDTs that can "switch gears" between
op-based and state-based modes.  For the lack of an establised term,
we call them *log-structured* RDTs.  These RDTs define the state as a
*compaction* of the log.  Other related terms are PORDT (pure
op-based) or ORDT (operational RDTs) Such data types fit well into
LSMT databases (LevelDB, Cassandra) and event-sourced storage systems
(e.g. Kafka).  Notably, these RDTs may need no explicit operation DAG.

Even further, we plan to consider a broad spectrum of replicated data
types with different properties.  For that, we use a formal RDT
framework (codenamed "ACID 2.0: associative, commutative, idempotent,
distributed", to be published).

Apart from a formal specification, we plan to ensure immediate
practical applicability of the results.  Regarding the computational
complexity and storage size, we use three relevant bounds:

    * `D` the size of the payload data,
    * `T` the maximum offline time allowed 
        (used with *data rotation rate* `r`),
    * `L` the maximum offline op log size (`L=TrD`).

Results are applicable if every algorithm and data structure is
`O(NlogN)` bounded, where `N` is one of `D`, `T`, `L` or a linear
combination thereof.  For example, due to the nature of log-structured
RDTs, a client's log size is `O(L)` and new peer join times are
`O(D)`, `O(D+L)`, and suchlike, depending on the RDTs used.

The planned protocol is based on the Replicated Object Notation (RON)
work.  The entirety of the protocol is subdivided into three layers:

1. Serialization format (Replicated Object Notation, text and binary
   variants).
2. A formal model for data types, a library of basic RDTs.
3. Replica's state machines (connection/disconnection cycle,
   query/response/update cycle, various replication modes, etc).

This protocol is sufficient to create compatible servers, clients, and
peers.  The user-facing API may vary between platforms.

For each layer, there are three deliverables:

1. A formal specification of the protocol allowing for compatible
   implementations (ideally, producing bitwise identical results);
2. key algorithms having memory/time complexity of O(NlogN); and
3. a reference implementation (go, C++, and/or JavaScript,
   fuzz-tested, MIT-licensed).

Hence, we have 9 parts (3 deliverables for each of 3 layers).

## Timeline

The overall planned rate is 1 deliverable a month.
The entire project is planned for 12 months.

## Budget

The project budget is $250K, 80% salaries, 20% travel and other
expenses.

