---
layout: default
title: Chains
in_section: specs
---

# Op chains

A chain is a group of ops from the same origin with consecutive id where each next op references the previous one.
Chains correspond to simple linear processes, like setting several fields of an object or typing in text.
While a chain is still a group of ops, RON adapts a shortcut notation for these, e.g.
<pre>
<font color="#6C6C6C">  1 </font><font color="#729FCF">@12345+origin</font> <font color="#3465A4">:prevop+origin</font> <font color="#8AE234"><b>&quot;Hello world&quot;</b></font>
</pre>
(Note double quotes, which is a notation for a chain of chars. Single quotes denote an atomic string.)
This chain unrolls into:
<pre>
<font color="#6C6C6C">  3 </font><font color="#729FCF">@12345+origin</font> <font color="#3465A4">:prevop+origin</font> <font color="#4E9A06">&apos;H&apos;</font>
<font color="#6C6C6C">  4 </font><font color="#729FCF">@1234500001+origin</font>           <font color="#4E9A06">&apos;e&apos;</font>
<font color="#6C6C6C">  5 </font><font color="#729FCF">@1234500002+origin</font>           <font color="#4E9A06">&apos;l&apos;</font>
<font color="#6C6C6C">  6 </font><font color="#729FCF">@1234500003+origin</font>           <font color="#4E9A06">&apos;l&apos;</font>
<font color="#6C6C6C">  7 </font><font color="#729FCF">@1234500004+origin</font>           <font color="#4E9A06">&apos;o&apos;</font>
<font color="#6C6C6C">  8 </font><font color="#729FCF">@1234500005+origin</font>           <font color="#4E9A06">&apos; &apos;</font>
<font color="#6C6C6C">  9 </font><font color="#729FCF">@1234500006+origin</font>           <font color="#4E9A06">&apos;w&apos;</font>
<font color="#6C6C6C"> 10 </font><font color="#729FCF">@1234500007+origin</font>           <font color="#4E9A06">&apos;o&apos;</font>
<font color="#6C6C6C"> 11 </font><font color="#729FCF">@1234500008+origin</font>           <font color="#4E9A06">&apos;r&apos;</font>
<font color="#6C6C6C"> 12 </font><font color="#729FCF">@1234500009+origin</font>           <font color="#4E9A06">&apos;l&apos;</font>
<font color="#6C6C6C"> 13 </font><font color="#729FCF">@123450000A+origin</font>           <font color="#4E9A06">&apos;d&apos;</font>
<font color="#6C6C6C"> 14 </font>
</pre>
Removals go in chains to, e.g. to remove the text above we send another chain:
<pre><font color="#6C6C6C"> 15 </font><font color="#729FCF">@23456+origin</font> <font color="#3465A4">:12345-origin</font> <font color="#FFD7D7"><b>(11)</b></font></pre>
(Note the reference has `-` instead of `+`. That means the tombstone flag is set, i.e. the referenced chain of 11 ops is removed.)
