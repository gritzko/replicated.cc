---
layout: default
title: LWW
in\_section: rdts
---

# LWW: Last-Write-Wins Object

LWW is the simplest & most popular distributed data type, but it isn't the most intelligent.
An LWW is a key-value object, with Unicode-encoded strings as keys, and any arbitrary [atoms](/specs/atoms) as values. Values may also be atom tuples.
The write with a later timestamp "wins", overriding any other operations that have happened to the same key before.
Nested objects are implemented as UUID references.

<pre>
<font color="#6C6C6C">  1 </font><font color="#A8A8A8"><b>@~</b></font> <font color="#4E9A06">&apos;an object is created&apos;</font><font color="#AF5F00">;</font>                                                                 
<font color="#6C6C6C">  2 </font><font color="#729FCF">@123+origin</font> <font color="#3465A4">:lww</font> <font color="#AF5F00">!</font>                                                                         
<font color="#6C6C6C">  3 </font>                                                                                           
<font color="#6C6C6C">  4 </font><font color="#A8A8A8"><b>@~</b></font> <font color="#4E9A06">&apos;set field `age` to value 18, set some other fields too:&apos;</font><font color="#AF5F00">;</font>                              
<font color="#6C6C6C">  5 </font><font color="#729FCF">@1234+origin</font> <font color="#3465A4">:123+origin</font> <font color="#4E9A06">&apos;age&apos;</font>    <font color="#87FFAF">18</font><font color="#AF5F00">;</font>                                                      
<font color="#6C6C6C">  6 </font>                         <font color="#4E9A06">&apos;order&apos;</font> <font color="#4E9A06">&apos;beer&apos;</font> <font color="#8AE234">0.5</font><font color="#AF5F00">;</font>                                               
<font color="#6C6C6C">  7 </font><font color="#A8A8A8"><b>@~</b></font> <font color="#4E9A06">&apos;see the chapter on chains on why the second record is missing id/ref&apos;</font><font color="#AF5F00">;</font>                 
<font color="#6C6C6C">  8 </font>                                                                                           
<font color="#6C6C6C">  9 </font><font color="#A8A8A8"><b>@~</b></font> <font color="#4E9A06">&apos;empty a field&apos;</font><font color="#AF5F00">;</font>                                                                        
<font color="#6C6C6C"> 10 </font><font color="#729FCF">@12345+origin</font> <font color="#3465A4">:1234000001+origin</font> <font color="#4E9A06">&apos;order&apos;</font> <font color="#AF5F00">;</font>                                                 
<font color="#6C6C6C"> 11 </font>                                                                                           
<font color="#6C6C6C"> 12 </font><font color="#A8A8A8"><b>@~</b></font> <font color="#4E9A06">&apos;the resulting state (no gc)&apos;</font>                                                           
<font color="#6C6C6C"> 13 </font><font color="#729FCF">@123+origin</font> <font color="#3465A4">:lww</font> <font color="#AF5F00">!</font>                                                                         
<font color="#6C6C6C"> 14 </font><font color="#729FCF">@1234+origin</font>    <font color="#4E9A06">&apos;age&apos;</font>    <font color="#87FFAF">18</font><font color="#AF5F00">,</font>                                                               
<font color="#6C6C6C"> 15 </font>                <font color="#4E9A06">&apos;order&apos;</font> <font color="#4E9A06">&apos;beer&apos;</font> <font color="#8AE234">0.5</font><font color="#AF5F00">,</font>                                                        
<font color="#6C6C6C"> 16 </font><font color="#729FCF">@12345+origin</font>   <font color="#4E9A06">&apos;order&apos;</font><font color="#AF5F00">,</font>                                                                   
<font color="#6C6C6C"> 17 </font>                                                                                           
<font color="#6C6C6C"> 18 </font><font color="#A8A8A8"><b>@~</b></font> <font color="#4E9A06">&apos;the resulting state (after gc)&apos;</font>                                                        
<font color="#6C6C6C"> 19 </font><font color="#729FCF">@123+origin</font> <font color="#3465A4">:lww</font> <font color="#AF5F00">!</font>                                                                         
<font color="#6C6C6C"> 20 </font><font color="#729FCF">@1234+origin</font>                      <font color="#4E9A06">&apos;age&apos;</font>    <font color="#87FFAF">18</font><font color="#AF5F00">,</font>                                             
<font color="#6C6C6C"> 21 </font><font color="#729FCF">@12345+origin</font> <font color="#3465A4">:1234000001+origin</font>  <font color="#4E9A06">&apos;order&apos;</font><font color="#AF5F00">,</font>                                                 
<font color="#6C6C6C"> 22 </font>                                                                                           
<font color="#6C6C6C"> 23 </font><font color="#A8A8A8"><b>@~</b></font> <font color="#4E9A06">&apos;undo the emptying (caveats apply - needs a no-gc version!)&apos;</font><font color="#AF5F00">;</font>                           
<font color="#6C6C6C"> 24 </font><font color="#729FCF">@123456+origin</font> <font color="#AD7FA8">:12345-origin</font> <font color="#AF5F00">;</font>     
</pre>


## Read next

[RGA/CT: Replicated Growable Array / Causal Tree](../rga/).
