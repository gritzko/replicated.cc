---
layout: default
title: Swarm API
in_section: swarm
---

# SwarmDB RON API


<pre>
<span class="line">  1 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;text snapshot dialog&apos;</span>
<span class="line">  2 </span><span class="rmid">@1object-id</span> <span class="ref">:txt</span> <span class="term">?</span>
<span class="line">  3 </span><span class="rmid">@1object-id</span> <span class="ref">:txt</span> <span class="string">&apos;here is the text&apos;</span> <span class="term">!</span>
<span class="line">  4 </span>
<span class="line">  5 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;real-time text subscription&apos;</span>
<span class="line">  6 </span><span class="rmid">@1object-id</span> <span class="ref">:txt</span> <span class="str_span"><b>live</b></span> <span class="term">?</span>
<span class="line">  7 </span><span class="rmid">@1object-id</span> <span class="ref">:txt</span> <span class="term">!</span>
<span class="line">  8 </span><span class="rmid">@1version-id</span>  <span class="string">&apos;here is the text&apos;</span><span class="term">,</span>
<span class="line">  9 </span><span class="rmid">@1nextver-id</span>  <span class="int">0</span> <span class="string">&apos;So, &apos;</span> <span class="int">8</span> <span class="int">-3</span> <span class="int">8</span> <span class="string">&apos;your&apos;</span><span class="term">,</span>
<span class="line"> 10 </span>
<span class="line"> 11 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;RON snapshot request&apos;</span>
<span class="line"> 12 </span><span class="id">@1version+id</span> <span class="ref">:rga</span> <span class="str_span"><b>live</b></span> <span class="term">?</span>
<span class="line"> 13 </span><span class="rmid">@1version-id</span> <span class="ref">:rga</span> <span class="term">!</span> <span class="term">,,,</span>
<span class="line"> 14 </span>
<span class="line"> 15 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;mapper write&apos;</span>
<span class="line"> 16 </span><span class="rmid">@1object-id</span> <span class="ref">:txt</span> <span class="term">!</span>
<span class="line"> 17 </span><span class="comment"><i>@0</i></span> <span class="derived_ref">:1nextver-id</span>  <span class="int">21</span> <span class="string">&apos;!&apos;</span><span class="term">,</span>
<span class="line"> 18 </span><span class="comment"><i>@00001</i></span> <span class="ref">:0</span> <span class="int">22</span> <span class="string">&apos;!!&apos;</span><span class="term">,</span>
<span class="line"> 19 </span>
<span class="line"> 20 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;mapper state write&apos;</span>
<span class="line"> 21 </span><span class="rmid">@1object-id</span> <span class="ref">:txt</span>  <span class="string">&apos;So, here is your text!&apos;</span><span class="term">;</span>
<span class="line"> 22 </span>
<span class="line"> 23 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;rga remove chain&apos;</span>
<span class="line"> 24 </span><span class="id">@1atime+orign</span> <span class="ref">:1alocatn+orign</span>  <span class="str_span"><b>(bs 3)</b></span><span class="term">;</span>
<span class="line"> 25 </span><span class="id">@1atime1+orign</span> <span class="ref">:1atime+orign</span>   <span class="str_span"><b>(un 3)</b></span><span class="term">;</span>
<span class="line"> 26 </span>
<span class="line"> 27 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;annotations&apos;</span>
<span class="line"> 28 </span><span class="rmid">@1atime1-orign</span> <span class="ref">:sha2</span> <span class="string">&apos;akbbvspoqvnoqnvqov...&apos;</span><span class="term">;</span>
<span class="line"> 29 </span>
<span class="line"> 30 </span><span class="comment"><b>@~</b></span> <span class="string">&apos;handshakes&apos;</span>
<span class="line"> 31 </span><span class="id">@1adBtiMe+orig</span> <span class="ref">:db</span> <span class="term">?</span>
<span class="line"> 32 </span><span class="id">@0+orig</span> <span class="ref">:db</span> <span class="str_span"><b>since</b></span> <span class="int">12345</span> <span class="term">?</span>
</pre>

