---
layout: default
title: UUIDs
section: uuids
---

# UUIDs

RON relies heavily on UUIDs to globally and unambigously identify its every construct: operations, objects, patches, versions, branches, snapshots etc. The RON UUID is one of several building blocks that lead to the immutable [op](/specs/ops).

Because RON employs UUIDs so heavily, it benefits greatly from using its own variant of UUID.
The RON UUID bit layout is backwards-compatible with [RFC 4122](https://tools.ietf.org/html/rfc4122) (128 bits, RFC4122-reserved flag values).
But unlike [RFC 4122](https://tools.ietf.org/html/rfc4122), time-based RON UUIDs can function as [Lamport/logical](https://en.wikipedia.org/wiki/Lamport_timestamps)/[hybrid](https://muratbuffalo.blogspot.com/2014/07/hybrid-logical-clocks.html) timestamps -- this nuance is critical for RON and CRDTs in general.

Additionally, the textual RON UUID serialization has some nice features too.

1. RON UUIDs are compact, thanks to Base64 and abbreviations. RON UUIDs are 23 chars max, typically less. RFC4122 is 43 chars of hex. MS GUIDs are 45 chars.
    Compare: `1fLDV+biQFvtGV` and `{G4G3G2G1-G6G5-G8G7-G9G10-G11G12G13G14G15G16}`.
2. RON UUIDs are meaningfully sorted lexicographically, thanks to the sortable variant of Base64.
3. Very much like RFC4122 UUIDs, RON UUIDs come in different versions: time-based event ids, time-based derived ids, hashes/numbers, and human-friendly string constants (names).
    For example, referencing a type of RON RDT can be done with a perfectly human-readable identifier: `lww`, `rga`, etc. The same applies to error identifiers (e.g. `NOTFOUND$~~~~~~~~~~`) and other global/transcendent constants.

To minimize the confusion, RON UUID bit layout is defined in terms of two 64-bit words.
The most significant word is *value* while the least-significant is *origin*.
Each word's most-significant four bits are flags, the rest is payload.
In accordance with RFC4122 (as much as it is possible), two most-significant bits of the origin are set to zero.
That corresponds to RFC4122 variant bits 00, thus overriding the RFC4122-reserved value "NCS backward compatibility".
We assume there are no [Apollo](https://en.wikipedia.org/wiki/Apollo_Computer) NCS UUIDs left in circulation.
The following two bits of the origin denote the RON UUID *version*.
Flag bits of the most-significant word denote the *variety*, see below.
Interpretation of the payload depends on the flag bits.
Most often, *value* is the actual value (e.g. a timestamp), while *origin* is a replica id.

<pre style="font-size: 80%">
value:   vvvv.... ........ ........ ........  ........ ........ ........ ........ 
origin:  00VV.... ........ ........ ........  ........ ........ ........ ........ 

flag bits:
  00    RFC4122 variant
  VV    version bits
  vvvv  variety bits
</pre>

## Flag bit coding

In the textual form, two version bits are encoded as a middle separator e.g. `@1hTDE6+test` or `NOTFOUND$~~~~~~~~~~`.


<ul class="nobullet">
  <li><code>$</code> for <code>00</code>: human readable names,</li>
  <li><code>%</code> for <code>01</code>: numbers and hashes,</li>
  <li><code>+</code> for <code>10</code>: events (Lamport timestamp, and origin),</li>
  <li><code>-</code> for <code>11</code>: derived events (same as event).</li>
</ul>

Four variety bits are encoded using single hex digit `0`..`F` followed by a slash `/`.
Variety of zero `0` can be omitted.

Interpretation of varieties depends on the version.


Varieties for version `$` (names) are:

<ul class="nobullet">
  <li><code>0000</code>: transcendental/hardcoded name (<code>lww</code>, <code>rga</code>) or a scoped name (<code>myvar$gritzko</code>),</li>
  <li><code>0001</code>: ISBN (<code>1/978$1400075997</code>,</li>
  <li><code>0011</code>: EAN-13 bar code (<code>3/4006381$333931</code>,</li>
  <li><code>0100</code>: SI units (<code>4/m</code>, <code>4/kg</code>,</li>
  <li><code>0101</code>: zip codes (<code>5/2628CD$NL</code>, <code>5/620078$RU</code>,</li>
  <li><code>1010</code>: IATA airport code (<code>A/LED</code>,</li>
  <li><code>1011</code>: ticker name (<code>B/GOOG$NASDAQ</code>,</li>
  <li><code>1100</code>: ISO 4217 currency code (<code>C/USD</code>, <code>C/GBP</code>,</li>
  <li><code>1101</code>: short DNS name (<code>D/google$com</code>,</li>
  <li><code>1110</code>: E.164 intl phone num (<code>E/7999$5631415</code>,</li>
  <li><code>1111</code>: ISO 3166 country code (<code>F/RU</code>, <code>F/FRA</code>...).</li>
</ul>

Varieties for version `%` (numbers and hashes) are:

<ul class="nobullet">
  <li><code>0000</code>..<code>0011</code>: Decimal index (up to <code>9999999999%</code>, also 2D indices <code>4%5</code>),</li>
  <li><code>0100</code>: SHA-2, plain chunk hash, first 120 bits,</li>
  <li><code>0101</code>: SHA-3, plain chunk hash,</li>
  <li><code>0110</code>: SHA-2 based RFC 7574 Merkle hash,</li>
  <li><code>0111</code>: SHA-3 based RFC 7574 Merkle hash,</li>
  <li><code>1000</code>..<code>1011</code>: Random number (<code>A/k3R9w_2F8w%Le~6dDScsw</code>),</li>
  <li><code>1100</code>..<code>1111</code>: Crypto id, public key fingerprint.</li>
</ul>

Varieties for versions `+` and `-` (events) are a combination of timestamp variety (m.s.bits) and origin variety:

Timestamp varieties are:

<ul class="nobullet">
  <li><code>00__</code>: Base64 calendar (<code>MMDHmSsnn</code>),</li>
  <li><code>01__</code>: Logical (<code>40000000001</code>),</li>
  <li><code>10__</code>: Epoch (RFC 4122 epoch, 100ns since 1582),</li>
</ul>

Origin varieties are:

<ul class="nobullet">
  <li><code>__00</code>: trie-forked,</li>
  <li><code>__01</code>: crypto-forked,</li>
  <li><code>__10</code>: record-forked,</li>
  <li><code>__11</code>: application-specific.</li>
</ul>

## UUID compression

Most of RON UUIDs can be efficiently compressed by only encoding highest significant bits.

Base 64×64 tailing zeroes can be omitted:

```
A/LED0000000+0000000000 = A/LED+0
A/LED0000000$123 = A/LED$123
```

If version is `$` and second component is zero, it can be fully omitted:

```
A/LED0000000$0000000000 = A/LED$0 = A/LED
```

If variety is `0`, it can be omitted as well:

```
0/lww0000000$0000000000 = lww0000000$0000000000 = lww$0 = lww
```

## See also

Integer encoding scheme: [Base 64×64](base64x64/).

Replica [assignments rules](replicas/).

Calendar-aware [timestamp encoding](timestamps/).
