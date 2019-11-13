---
layout: default
title: Frames
in_section: specs
---

# Frames

A frame is a collection of ops (operations) applied/executed atomically: either all or nothing.
A frame may contain a multitude of [chunks](/specs/glossary#chunk) - object states, separate ops, patches, queries and so on.

Frames can’t be split (but they can be joined together).

In the textual RON, frames end with a `.` dot.
Chunks within a frame end with their respective separators:

*  a semicolon `;` for event chunks,
*  an exclamation mark `!` for assertions/new writes, or
*  a question mark `?` for query chunks.


## Event chunks

As RON is an [event-sourced](https://martinfowler.com/eaaDev/EventSourcing.html) format, any relay of data is made by sending some collection of immutable ops.
These ops express certain events that already happened.
The final state of a system is a [reduction](https://en.wikipedia.org/wiki/Fold_\(higher-order_function\)) of those ops.

Those event ops may come in different groupings.
Before any reduction, these are separate raw ops, or spans and chains of raw ops.
After a reduction, ops form either an object state or a patch or some combination of these.

A minimal object state  would consist of just a header (the object creation op):
<pre><span class="line">  1 </span><span class="id">@1D4ICCA+XU5eRJ</span> <span class="span">:lww</span><span class="term">;</span> <span class="full_stop">.</span>
</pre>

Here is an example of a frame containing one chunk, a state of a Last-Write-Wins object:

<pre><span class="line">  1 </span><span class="id">@1D4ICCA+XU5eRJ</span> <span class="span">:lww</span><span class="term">,</span>
<span class="line">  2 </span>     <span class="str_span">&apos;x&apos;</span> <span class="int">356</span><span class="term">,</span>
<span class="line">  3 </span>     <span class="str_span">&apos;y&apos;</span> <span class="int">83</span><span class="term">;</span>
<span class="line">  4 </span><span class="full_stop">.</span>
</pre>


## Assertion chunks

Assertions are events as they happen, before they get serialized as immutable ops.
Asserions are only used between a RON replica and some external entity.
Effectively, that is a part of the external API of a RON system.
While being serialized as ops, assertions don't have event UUIDs, they are not immutable, they are not relay-able.

Suppose, we'd like to correct a text "Helo world." into "Hello world!".
Internally, that text is kept as a RON RGA object.
We don't like to fiddle with RGA, so we make a write through a `txt` mapper by issuing an assertion:
<pre><span class="line">  1 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">patch</span><span class="term">,</span> <span class="int">3</span> <span class="str_span">&apos;l&apos;</span> <span class="int">10</span> <span class="int">-1</span> <span class="int">10</span> <span class="str_span">&apos;!&apos;</span> <span class="term">!</span>
</pre>

As you might see, we order a deletion at position 10 and two insertions, at positions 3 and 10.
That might be a risky practice, actually.
We sent a patch, but we only mentioned the object id, not the object version.
If there was any concurrent change, our offsets might be off.
Probably, we should do it like this:

<pre><span class="line">  1 </span> <span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">&apos;Hello world!&apos;</span> <span class="term">!</span>
</pre>

Well, if there was any concurrent change, we just overwrote it.
Screw them.

OK, let's try it once again, but this time we mention the exact version for the offsets:

<pre><span class="line">  1 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">patch at 1l55ku+test</span><span class="term">,</span> <span class="int">3</span> <span class="str_span">&apos;l&apos;</span> <span class="int">10</span> <span class="int">-1</span> <span class="int">10</span> <span class="str_span">&apos;!&apos;</span> <span class="term">!</span>
</pre>

Should be OK... unless someone did exactly the same changes concurrently... or slightly different changes maybe. Either way, RON does not guarantee semantic convergence because that would require a [Turing-test](https://en.wikipedia.org/wiki/Turing_test) capable solver. RON only guarantees replicated data type convergence, so use it wisely.

## Query chunks

Suppose, we'd like to read the hello-world text mentioned above.
Then, we should issue a RON query:

<pre><span class="line">  1 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="term">?</span>
</pre>

In SwarmDB, a query is evaluated in the context of its frame, including all the effect of all the preceding ops.

## Compression

Within a frame, ops might be compressed.
Various RON serializations may employ various compression tricks.
The textual RON employs chain compression:

1. the reference UUID is skipped if it matches the id of the previous op,
2. the id of the op is skipped if it is an increment of the the previous op id.

This way, op spans only mention UUIDs in the first op, others are skipped.

In this example again, a frame contains a single chunk which is an object state consisting of a single three-op span.
The header mentions the event id `1D4ICCA+XU5eRJ` and a reference id `lww`. The other two ops don't.

<pre><span class="line">  1 </span><span class="id">@1D4ICCA+XU5eRJ</span> <span class="span">:lww</span><span class="term">,</span>
<span class="line">  2 </span>     <span class="str_span">&apos;x&apos;</span> <span class="int">356</span><span class="term">,</span>
<span class="line">  3 </span>     <span class="str_span">&apos;y&apos;</span> <span class="int">83</span><span class="term">;</span>
<span class="line">  4 </span><span class="full_stop">.</span>
</pre>

## Read next

[Nominal format](../nominal).
