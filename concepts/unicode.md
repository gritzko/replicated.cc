# 30 years in, is Unicode a disaster?

RON is a data synchronization protocol that must guarantee *bitwise* convergence of replicas.
One frequently asked question was why don't we use JSON for that.
["Parsing JSON is a Minefield"](http://seriot.ch/parsing_json.php) is an excellent post by Nicolas Seriot explaining why we can't.
Various JSON specification subtleties led to different parsers perceiving inputs differently.
If we use JSON as the carrying format, we inherit all these issues.
That might turn difficult or impossible to solve.
JSON is lax.

But this rabbit hole is so much deeper!

The same post lists [various issues](http://seriot.ch/parsing_json.php#25) JSON inherited from Unicode.
First of all, there are many Unicode encodings out there.
For example, your file is UTF-8 but your JavaScript runtime uses UTF-16 because it needs fixed-width characters.
Each Unicode encoding has its peculiarities.
For example, UTF-8 allows for [overlong encodings](https://en.wikipedia.org/wiki/UTF-8#Overlong_encodings).
Proper UTF-8 sequences may encode codepoints which are not valid Unicode.
UTF-16 uses *surrogates* to encode codepoints beyond the [Basic Multilingual Plane (BMP)](https://en.wikipedia.org/wiki/Plane_(Unicode)#Basic_Multilingual_Plane).
There is always a possibility a file might have a [byte order mark (BOM)](https://en.wikipedia.org/wiki/Byte_order_mark).
Java tends to use [*modified* UTF-8](https://en.wikipedia.org/wiki/UTF-8#Modified_UTF-8).
Unicode encodings may use different endianness, etc etc.

As a result, RON uses UTF-32 strings to define its hashing functions.
Other forms are just too lax!
For codings other than UTF-32, we can't reasonably expect every implementation to produce bit-precise results, especially if we consider various adversarial scenarios.
There are way too many subtleties.
If we recode a RON snippet from binary to text, reformat it, send it over network, store in binary, read it back, then the result must verify to exactly the same SHA hash.
Unfortunately, we can not mandate some single coding: we need binary and text, different forms of storage and different forms of compression all to be available.

In turn, UTF-32 hashing makes it necessary to recode all strings for integrity checking.
That is in addition to any recoding the underlying runtime may need (e.g. UTF-8 <-> UTF-16).

Note that the only thing we are trying to achieve here is a bitwise reliable serialization.
If we'd like to render or edit that Unicode text, that is a [much bigger topic](https://lord.io/blog/2019/text-editing-hates-you-too/).

Initially, Unicode was envisioned as a simple fixed-width two-byte-per-character coding, see [UCS-2](https://en.wikipedia.org/wiki/Universal_Coded_Character_Set).
With such a coding, there is not much space left for subtleties.
It was the version 2.0 in 1996 when the hell broke loose.
Unicode 2.0 extended beyond two-byte BMP thus bringing us a multitude of encodings, all with their pitfalls and subtleties and conversions and signalling and everything else.

Given all of the above, I wonder: was Unicode 2.0 worth it at all?
Was it really necessary to make Unicode *extensible*?
To get some hard numbers on that, I implemented a small UTF-8 parser in Ragel and scanned several Wikipedia dumps (English en, Russian ru, Chinese zh, Japanese ja).

```
==> en-count-128.txt <==
ASCII   symbols 75671324619
BMP     symbols 194376047
nonBMP  symbols 214552
invalid symbols 0
all     symbols 75865915218
broken  symbols 0
total   bytes   76154026008


==> ja-count-128.txt <==
ASCII   symbols 4964998080
BMP     symbols 2615045414
nonBMP  symbols 10159
invalid symbols 0
all     symbols 7580053653
broken  symbols 0
total   bytes   12803783970


==> ru-count-128.txt <==
ASCII   symbols 8578729765
BMP     symbols 6996711794
nonBMP  symbols 35206
invalid symbols 0
all     symbols 15575476765
broken  symbols 0
total   bytes   22606207698


==> zh-count-128.txt <==
ASCII   symbols 5084168153
BMP     symbols 1071380994
nonBMP  symbols 62854
invalid symbols 0
all     symbols 6155612001
broken  symbols 0
total   bytes   8287802079

```

As we see, the use of non-BMP codepoints is *FIVE ORDERS OF MAGNITUDE LOWER!!!*
So no, people don't need it that much.
That is easy to comprehend, as non-BMP planes mostly contain extinct languages and other very-rarely-used things.
There are also Emojis, but Wikipedia is not rich in those, for obvious reasons.

That is extremely annoying to see!
The [Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle) says 20% cost gives 80% value.
Well.
With Unicode, it is more like 99.999% of costs gives 0.001% of value. Or vice-versa.

Today's Unicode supports Emoji.
Some may question the wisdom of bundling Emojis into Unicode as their coloring and clustering logic differs drastically from what alphabets and scripts normally do.
Skipping that, the Unicode Consortium site [lists 1680 emojis people actually use](https://home.unicode.org/emoji/emoji-frequency/).
That is less than the size of the private-use BMP range: F8FF-E000=18FF or 6400 in decimal.
If we need some frivolous use cases in messenger apps, why don't we declare it private-use, I wonder?

Unicode supports Gothic and Linear B.
We bear the costs, but who benefits from that?
We can't even say it is some vocal minority because they are no longer vocal, for thousands of years and counting.
So, what was the point?
Scripts and languages are easy to design; JRR Tolkien invented several, just for fun.
But, they are insanely expensive to support: people must use them to keep them afloat.
[The meaning of a word is its use in the language.](https://philosophyforchange.wordpress.com/2014/03/11/meaning-is-use-wittgenstein-on-the-limits-of-language/)
Thousands of new characters will not appear out of nowhere. 
You have to wait another thousand years for that!

So, what can we do about it all?
We can not change Unicode.
That ship has sailed in 1996, I guess.
Still, what conclusions can we make?

Unicode belongs to the same family of interoperability standards as IP or HTTP or Ethernet.
For such a standard to work, everyone in the world must be able to read and write it.
Because that's what it's for!
Such a standard must be 80/20 *by design* because it imposes costs on basically everyone.
For every feature, we should ask whether it is worth imposing this cost *on everybody in the world*.

Exactly the same logic applies to any interoperability standard.
**A good standard is 80/20 and final.**

Consider Ethernet.
It was blasted by critics for its simplicity - and it won the world.
IPv4? Same story.
Or, consider HTTP 1.1.
Very simple, won the world.
Was HTTP/2 better?
Well, it was not simpler, for sure.
Some say, it makes the Web an extension of the Google's internal RPC system.
Endpoints of the Web are increasingly controlled by just a few entities, so interoperability may no longer be the topmost priority anymore.
These days, we have two browsers left and that may not be the final number.
Even Microsoft can not afford to maintain one!

Another take: extending an interoperability standard means sabotaging it.

Still, I wonder whether we can move in the opposite direction at all?
Let's call it the SCRU principle: Support the Critical, Reject the Unnecessary.
*SCRU Unicode 1.0* is BMP in two-byte fixed-width little-endian coding.
End of the spec.

