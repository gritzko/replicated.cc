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
Every RON object may naturally have an unlimited number of replicas that may
synchronize incrementally, mostly in real-time. 
Every object, every change, every version has a globally unique UUID.
Thanks to CRDTs and UUIDs, all changes merge and converge correctly.

Here is a simple object serialized in RON:

<pre>
<font color="#6C6C6C">  1 </font><font color="#729FCF">@23456+thinkpad</font> <font color="#3465A4">:lww</font> <font color="#AF5F00">!</font>
<font color="#6C6C6C">  2 </font>        <font color="#4E9A06">&apos;id&apos;</font>            <font color="#4E9A06">&apos;20MF000CUS&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  3 </font>        <font color="#4E9A06">&apos;type&apos;</font>          <font color="#4E9A06">&apos;laptop&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  4 </font>        <font color="#4E9A06">&apos;cpu&apos;</font>           <font color="#4E9A06">&apos;i7-8850H&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  5 </font>        <font color="#4E9A06">&apos;display&apos;</font>       <font color="#4E9A06">&apos;15.6” UHD (3840 x 2160) IPS multi-touch, anti-reflective, 400nits&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  6 </font>        <font color="#4E9A06">&apos;RAM&apos;</font>           <font color="#4E9A06">&apos;16 GB DDR4 2666MHz&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  7 </font>        <font color="#4E9A06">&apos;storage&apos;</font>       <font color="#4E9A06">&apos;512 GB Solid State Drive, PCIe-NVME OPAL2.0 M.2&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  8 </font>        <font color="#4E9A06">&apos;graphics&apos;</font>      <font color="#4E9A06">&apos;NVIDIA GeForce GTX 1050Ti 4GB&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C">  9 </font><font color="#729FCF">@34567+thinkpad</font>
<font color="#6C6C6C"> 10 </font>        <font color="#4E9A06">&apos;wlan&apos;</font>          <font color="#4E9A06">&apos;Intel 9560 802.11AC vPro (2 x 2) &amp; Bluetooth 5.0&apos;</font><font color="#AF5F00">,</font>
<font color="#6C6C6C"> 11 </font>        <font color="#4E9A06">&apos;camera&apos;</font>        <font color="#4E9A06">&apos;IR &amp; 720p HD Camera with microphone&apos;</font><font color="#AF5F00">,</font>
</pre>

Key RON principles are:

- **Immutability**. Once created RON operations travel through system unchanged.
- **Addressability**. Everything: changes, versions, every piece of data is uniquely identified and globally referenceable.
- **Source and transport agnosticism**. It doesn’t matter where RON data comes from and how. Any node can read RON from the network, filesystem, database, message bus and/or local cache, in any order, and merge them correctly.
- **Efficiency**. RON data is optimized to make metadata overhead tolerable and real-time collaborative applications possible. It’s optimized machine first, human readability second.
- **Causality**. Each RON operation explicitly references what version of an object it is based on. No matter how and when you get your data, you can always reconstruct the correct order events have happened in.

RON is an answer to the new reality where different agents join and leave network all the time and communicate with each other without any order or reliability guarantees. In other words, RON is lingua franca for Generation Internet.


For explanation, see [UUIDs](/uuids/) and [specification](/specs/).

