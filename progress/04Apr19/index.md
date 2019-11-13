# RFP 5 "RON" 6-month progress report

CRDT is a data replication/synchronization technology that needs no central
"master" replica. As such, it is often considered an essential building
block for distributed/decentralized systems.  While the core principles of
CRDT are clear, there is certainly a gap between theory and practice. Our
research addresses the gap by focusing on the following topics:

1. log-structured (aka operational) CRDTs; those can "switch gears" between
op-based, patch-based and state-based modes of operation (those correspond
to real-time sync, periodic sync and full reconciliation).
2. minimizing metadata overhead by employing chain structures; the effect
is similar to the block-based approach, except the core model stays simple.
3. a formal ACID 2.0 framework to mix different RDTs in the same
environment (ACID ~ associative, commutative, idempotent, distributed).
4. finally, unifying Causal and Merkle structures (CRDTs and crypto).

That work is divided into three layers:

1. the foundation: an op-based formal data model and protocol (Replicated
   Object Notation),
2. replicated data types (RDTs) based on the RON model,
3. on top of that, the (database) replication part.

For each of the layers, we provide three deliverables:

1. a formal specification,
2. algorithms with pragmatic complexity bounds, and
3. a reference implementation.

Overall, the aim is to make CRDTs a simple universal instrument within the
reach of every developer. Our work is online at
[http://replicated.cc](http://replicated.cc).

## Progress report

```
+--------------+---------------+---------------+---------------+
| Guesstimates |      RON      |      RDT      |    replica    |
+--------------+---------------+---------------+---------------+
|specification |      85%      |      50%      |      50%      |
+--------------+---------------+---------------+---------------+
|algorithms    |      100%     |      75%      |      50%      |
+--------------+---------------+---------------+---------------+
|implementation|      50%      |      50%      |      25%      |
+--------------+---------------+---------------+---------------+
```

1. Replicated Object Notation: stable. The notation is subdivided into the
[nominal format](http://replicated.cc/specs/nominal) and various
serialization formats. The nominal format is zero-liberty, bit-precise.
Algorithms and hashes are defined on that. Serializations focus on
practical issues: compression and performance (the binary format), human
readability (text) or compatibility (JSON/CBOR).

    * RON-spec: the nominal format is finished, frozen. The text based
      format is mainly finished, short of some optimizations; the
      [formal grammar](https://github.com/gritzko/ron-cxx/blob/master/ragel/text-grammar.rl)
      is frozen either way. The binary format still needs work,
      JSON/CBOR formats are merely outlined.
    * RON-algo: finished, frozen (serialization-independent).
    * RON-code: API is done, the text-based variant of the protocol is done
      and frozen (fuzz-tested). The rest of the code is made in a way to
      make other serializations (binary, JSON/CBOR) pluggable.

2. Replicated Data Types: basic RDTs are finished, frozen.

    * RDT-spec: some RDTs had to be reworked to make them chain compression
      friendly. That is partly [documented](http://replicated.cc/rdts/rga).
      Although, a proper formal bit-precise specification is future work.
    * RDT-algo: finished parts are frozen. The key element is the `O(NlogN)`
      iterator-heap merge algorithm.
    * RDT-code: a variety of RDTs still need to be implemented.

3. Replica: theory solved, implementation in progress.

    * Replica-spec: some parts are stable (the key-value storage layout,
      connection state machine). The branch machinery is reasonably stable.
    * Replica-algo: patch production and data deduplication will need some
      work, the rest is reasonably stable.
    * Replica-code: [progressing](https://github.com/gritzko/ron-cxx/db/).
      The implementation is done in C++ for two reasons. First, to
      integrae with RocksDB which does all the heavylifting for us.
      Second, to make bindings to higher level languages the usual way.

Informally speaking, the goal of the first half of the project was to make
our LEGO bricks; a manageable coherent codebase we can build upon.  In the
second half, we build with those bricks, focusing on use-case testing and
performance optimization.  At this point, a sufficient set of bricks is
complete and "frozen".  Any further additions to the RON/RDT part are
either non-critical features (add more RDTs) or performance optimizations
(binary serialization, chain span compression, etc).

While the general framework of log-structured CRDTs was clear, a lot of
work went into re-adjusting primitives to address such long-standing
problems as CRDT metadata size and merge algorithm performance.  That
necessitated readjustment of other elements further up the stack (e.g. the
Causal Tree data struture, key-value storage layout). Our reward here is a
breed of RDTs with conflict-free `O(NlogN)` merge, where big-O does not
treacherously hide a constant too big. Such RDT merges are truly
frictionless, so we may run them in the core of a database.

Blending CRDTs and Merkle structures was a separate line of work.
Bit-precise convergence requires a very strict and formal approach to
everything. Hence we introduced the nominal format, Unicode handling and
endianness rules, et cetera.  That was plenty of tedious work. The
resulting technique is rather novel: the crypto DAG is defined as an
*overlay* of the data DAG; the data itself does not explicitly include
hashes or other crypto primitives. That was necessitated by the fact our
data graph is very fine-grained (op-based). We can not afford to keep all
the historical hashes and signatures; that is way too much of
incompressible data. Ideally, we want to keep the latest entries only,
everything redundant being dropped ASAP. As long as it is convenient to
re-calculate the hashes, their explicit storage is unnecessary.  (That does
not prevent us from armoring any outer links with content hashes!)

One new exciting thing that is still evolving is *branches*.  It is an
unified approach that generalizes branches (in the distributed version
control sense) and dataset federation. That was only made possible by
the frictionless merge we discussed earlier.

A significant part of the work is *mappers*. Those are stateless functions
converting between RON and other formats (plain text, CSV, JSON). As
mappers encapsulate all the CRDT/RON machinery, they greatly simplify the
creation of APIs and bindings for various languages and platforms
(priorities: golang, JavaScript/node.js and JavaScript/WebAssembly; there
is some ongoing Haskell and Rust experimentation too).

At this point, the focus shifts to use case testing, as any practical
application of the technology provides valuable feedback and sets
reasonable priorities. In that regard, we have one priority objective, one
objective and one non-objective.

1. The priority is a ["CRDT-based revision control
  system"](http://replicated.cc/swarm/dvcs). We don't plan to dethrone git
  yet, but we definitely want to learn strong and weak sides of the
  technology in this regard. If git is a "content-centric filesystem with
  a VCS interface", we are creating a "CRDT key-value store with a VCS
  interface".
2. Another objective is a "federated package database", also described as
  "the npm database without npm, Inc". As package metadata obeys the
  package dependency DAG, it is perfectly feasible to retrieve and process
  that data in a decentralized fashion. (The one hosting the package
  should cache its dependencies too. If the data is verifiable,
  one needs neither recursive retrieval nor central database.)
3. A clear non-objective is a standalone database with a query language.
  Any work in this direction is a no-no, simply to avoid de-focusing.
  We do a syncable embedded key-value store.

Either way, we will seek community feedback by making a pre-release ASAP.
We will do our best to shorten the lag between the actual code and the
documentation on http://replicated.cc.

Thanks for reading!

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
