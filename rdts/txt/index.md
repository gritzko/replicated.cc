---
layout: default
title: TXT
in_section: rdts
---

# TXT: Collaborative text

`txt` is a plaintext mapper for the `rga` RDT.
It converts RGA arrays to human-readable text, assuming the RGA contains chars (not numbers or strings or UUIDs).

Here is an example conversation with the txt mapper:

<pre><span class="line">  1 </span><span class="comment">@~</span> <span class="str_span">&apos;create a CT&apos;</span> <span class="term">!</span>
<span class="line">  2 </span><span class="id">@1l54hK+test</span> <span class="span">:rga</span><span class="term">,</span> <span class="str_span">&apos;w&apos;</span><span class="term">,</span> <span class="str_span">&apos;o&apos;</span><span class="term">,</span> <span class="str_span">&apos;r&apos;</span><span class="term">,</span> <span class="str_span">&apos;l&apos;</span><span class="term">,</span> <span class="str_span">&apos;d&apos;</span> <span class="term">;</span>
<span class="line">  3 </span>
<span class="line">  4 </span><span class="comment">@~</span> <span class="str_span">&apos;read the text&apos;</span> <span class="term">!</span>
<span class="line">  5 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span><span class="term">?</span>
<span class="line">  6 </span>
<span class="line">  7 </span><span class="comment">@~</span> <span class="str_span">&apos;text is OK&apos;</span><span class="term">?</span>
<span class="line">  8 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">&apos;world&apos;</span> <span class="term">;</span>
<span class="line">  9 </span>
<span class="line"> 10 </span><span class="comment">@~</span> <span class="str_span">&apos;write the state&apos;</span> <span class="term">!</span>
<span class="line"> 11 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">&apos;Helo world.&apos;</span> <span class="term">!</span>
<span class="line"> 12 </span><span class="comment"><i>@0</i></span> <span class="span">:tag</span> <span class="string">AT~HELO</span> <span class="term">!</span>
<span class="line"> 13 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="term">?</span>
<span class="line"> 14 </span>
<span class="line"> 15 </span><span class="comment">@~</span> <span class="str_span">&apos;check the state&apos;</span> <span class="term">?</span>
<span class="line"> 16 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">&apos;Helo world.&apos;</span><span class="term">;</span>
<span class="line"> 17 </span>
<span class="line"> 18 </span><span class="comment">@~</span> <span class="str_span">&apos;a patch&apos;</span> <span class="term">!</span>
<span class="line"> 19 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">patch</span><span class="term">,</span>
<span class="line"> 20 </span>    <span class="int">3</span> <span class="str_span">&apos;l&apos;</span> <span class="int">9</span> <span class="int">-1</span> <span class="int">10</span> <span class="str_span">&apos;!&apos;</span> <span class="term">!</span>
<span class="line"> 21 </span>
<span class="line"> 22 </span><span class="comment">@~</span> <span class="str_span">&apos;query the patched text&apos;</span><span class="term">!</span>
<span class="line"> 23 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="term">?</span>
<span class="line"> 24 </span>
<span class="line"> 25 </span><span class="comment">@~</span> <span class="str_span">&apos;is it patched OK?&apos;</span> <span class="term">?</span>
<span class="line"> 26 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">&apos;Hello world!&apos;</span><span class="term">;</span>
<span class="line"> 27 </span>
<span class="line"> 28 </span><span class="comment">@~</span> <span class="str_span">&apos;query the old version&apos;</span><span class="term">!</span>
<span class="line"> 29 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">at</span> <span class="string">AT~HELO</span> <span class="term">?</span>
<span class="line"> 30 </span>
<span class="line"> 31 </span><span class="comment">@~</span> <span class="str_span">&apos;is the old version recovered?&apos;</span> <span class="term">?</span>
<span class="line"> 32 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">at</span> <span class="string">AT~HELO</span><span class="term">,</span>
<span class="line"> 33 </span>    <span class="str_span">&apos;Helo world.&apos;</span><span class="term">;</span>
<span class="line"> 34 </span>
<span class="line"> 35 </span><span class="comment">@~</span> <span class="str_span">&apos;query a diff&apos;</span><span class="term">!</span>
<span class="line"> 36 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">since</span> <span class="string">AT~HELO</span> <span class="term">?</span>
<span class="line"> 37 </span>
<span class="line"> 38 </span><span class="comment">@~</span> <span class="str_span">&apos;check the diff&apos;</span><span class="term">?</span>
<span class="line"> 39 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">since</span> <span class="string">AT~HELO</span><span class="term">,</span>
<span class="line"> 40 </span>    <span class="int">3</span> <span class="str_span">&apos;l&apos;</span> <span class="int">9</span> <span class="int">-1</span> <span class="int">10</span> <span class="str_span">&apos;!&apos;</span> <span class="term">;</span>
<span class="line"> 41 </span>
<span class="line"> 42 </span><span class="comment">@~</span> <span class="str_span">&apos;query a hili&apos;</span><span class="term">!</span>
<span class="line"> 43 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">hili</span> <span class="string">AT~HELO</span> <span class="term">?</span>
<span class="line"> 44 </span>
<span class="line"> 45 </span><span class="comment">@~</span> <span class="str_span">&apos;check the hili&apos;</span><span class="term">?</span>
<span class="line"> 46 </span><span class="derived_id">@1l54hK-test</span> <span class="span">:txt</span> <span class="str_span">hili</span> <span class="string">AT~HELO</span><span class="term">,</span>
<span class="line"> 47 </span>    <span class="str_span">eq</span> <span class="str_span">&apos;Hel&apos;</span> <span class="str_span">in</span> <span class="str_span">&apos;l&apos;</span> <span class="str_span">eq</span> <span class="str_span">&apos;o world&apos;</span> <span class="str_span">in</span> <span class="str_span">&apos;!&apos;</span> <span class="str_span">rm</span> <span class="str_span">&apos;.&apos;</span><span class="term">;</span>
<span class="line"> 48 </span>
</pre>

## Read next

[Sets](../set/).
