# RON term glossary

<img style="float: right;" src="dna.png">

* RON UUID - a 128-bit globally unique identifier, one of four *versions*:
    * time-based (a logical/hybrid timestamp, 60 bits of timestamp, 60 bits of event *origin* id)
    * name (a human readable name of some predefined concept)
    * numeric (either a random number or a hash)
* Atom - an immutable value of one of four types:
    * RON UUID
    * Integer (64-bit signed)
    * String (UTF-8)
    * Floating-point number (IEEE 754-2008, 64 bit)
* Op - an immutable unit of change, consists of:
    * specifier (metadata, key)
        * own RON UUID (id, identifies the op)
        * reference RON UUID (ref, identifies the op's location in the data graph)
    * value
        * any number (zero or more) of value atoms (UUIDs, ints, strings, floats)

Most other higher-value constructs are composed of ops:

* Chain - a sequence of ops from the same origin, where each next op references the previous one
          (in an *even* chain, each op id is the previous id + 1)
* Yarn - a linear log of all ops from the same origin (corresponds to a Lamport process)
* Tree - a causally ordered group of ops forming a tree (each next op references some preceding op from the tree, except for the root op)
* Object - largely synonymous to a tree, although op ordering depends on the data type (RDT)
* Patch - a group of ops modifying the same tree/object (causally consistent, i.e. referencing the existing ops of the tree or previous ops of the patch)
* Frame - largely synonymous to a "write batch"; a group of ops to be applied in a single transaction
* Chunk
* Log - a causally ordered sequence of ops, like a database op log
* Graph - a group of objects referencing each other
* Graph patch - a group of object patches and full object states, a causally consistent change of an object graph
