---
layout: default
title: Replicated Object Notation
---

# Replicated Object Notation

Replicated Object Notation (RON) is a format for *distributed live data*. 
RON's primary mission is continuous data synchronization.

JSON, protobuf, and others, all imply serialization of state snapshots.
RON handles state and updates all the same: _state is change and change is state_.

RON has versioning and addressing, all built in.
Every RON object may naturally have any number of replicas,
which may synchronize in real-time or live offline. 
Every object, every change, every version has a globally unique UUID.
Thanks to CRDTs and UUIDs, all changes merge and converge consistently.

Here is a simple object serialized in RON:

<pre>
<font color="#6C6C6C">  1 </font><font color="#729FCF">@1fLDV+biQFvtGV</font> <font color="#3465A4">:lww</font> <font color="#AF5F00">!</font>
<font color="#6C6C6C">  2 </font>        <font color="#4E9A06">&apos;id&apos;</font>            <font color="#4E9A06">&apos;20MF000CUS&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  3 </font>        <font color="#4E9A06">&apos;type&apos;</font>          <font color="#4E9A06">&apos;laptop&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  4 </font>        <font color="#4E9A06">&apos;cpu&apos;</font>           <font color="#4E9A06">&apos;i7-8850H&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  5 </font>        <font color="#4E9A06">&apos;display&apos;</font>       <font color="#4E9A06">&apos;15.6” UHD IPS multi-touch, 400nits&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  6 </font>        <font color="#4E9A06">&apos;RAM&apos;</font>           <font color="#4E9A06">&apos;16 GB DDR4 2666MHz&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  7 </font>        <font color="#4E9A06">&apos;storage&apos;</font>       <font color="#4E9A06">&apos;512 GB SSD, PCIe-NVME M.2&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  8 </font>        <font color="#4E9A06">&apos;graphics&apos;</font>      <font color="#4E9A06">&apos;NVIDIA GeForce GTX 1050Ti 4GB&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  9 </font>        <font color="#729FCF">@1fLDk4+biQFvtGV</font>
<font color="#6C6C6C"> 10 </font>        <font color="#4E9A06">&apos;wlan&apos;</font>          <font color="#4E9A06">&apos;Intel 9560 802.11AC vPro&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C"> 11 </font>        <font color="#4E9A06">&apos;camera&apos;</font>        <font color="#4E9A06">&apos;IR &amp; 720p HD Camera with microphone&apos;</font><font color="#AF5F00">,</font>
<pre><font color="#6C6C6C"> 12 </font>        <font color="#A8A8A8"><i>@sha3</i></font> <font color="#4E9A06">&apos;SfiKqD1atGU5xxv1NLp8uZbAcHQDcX~a1HVk5rQFy_nq&apos;</font><font color="#AF5F00">,</font>
</pre>

Key RON principles are:

- **Immutability** - RON sees data as a collection of immutable timestamped ops. In the example above, every key-value pair is an op,
        which may be addressed, transmitted, stored, applied or rolled back, garbage collected, etc etc.
        Both the object's state and any changes are collections of immutable ops.
- **Integrity** - ops form a Merkle tree, so the data is integrity-checked to the last bit, if necessary (like in git).
        In the example, ten ops form a Merkle chain, so the hash of the last op covers them all.
- **Addressability**. Everything: changes, versions, every piece of data is uniquely identified and globally referenceable.
        Above, the first op has an id `1fLDV00000+biQFvtGV`, the second one is `1fLDV00001+biQFvtGV`, and so on.
        The last two ops belong to a different changeset, so their ids are `1fLDk40000+biQFvtGV`, `1fLDk40001+biQFvtGV`.
- **Causality**. Each RON operation explicitly *references* what version of an object it is based on.
        No matter how and when you get your data, you can always reconstruct the correct order and location of data pieces.
- **Efficiency**. RON data is optimized to make metadata overhead bearable.
        As an op is a very fine-grained unit of change, RON optimizes per-op metadata overhead in numerous ways.
        For example, both op ids and references are skipped if they go incrementally. In the example above, the second
        op mentions neither its own id (the first plus 1) nor its reference (the first op).

RON is an answer to the new reality: swarms of mobile devices communicating over unreliable wireless networks in an untrusted environment.

For more in-depth reading, please see an explanation of [RON UUIDs](/uuids/) and the [protocol specification](/specs/).
[RON RDTs](/rdts/) is a convergent variant of [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type)
able to operate in different modes (op-based, state-based, delta-enabled).
[SwarmDB](/swarm/) is a reference implementation of a RON-based syncable key-value store.

