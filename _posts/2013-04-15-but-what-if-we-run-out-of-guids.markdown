---
layout: post
title: But What If We Run Out of Guids?
tags: [humor, programming]
---

There was a bit of programmer humor happening on Twitter today concerning GUIDs.

<blockquote class="twitter-tweet" data-conversation="none" lang="en">
   <p>
      <a href="https://twitter.com/reverentgeek">@reverentgeek</a> 
      <a href="https://twitter.com/digitalBush">@digitalBush</a> Guys, will you please STOP generating GUIDs for jokes? You&#39;re just wasting them! STOP WASTING GUIDS <a href="https://twitter.com/search?q=%23saveTheGuids&amp;src=hash">#saveTheGuids</a>
   </p>&mdash; Robert Simmons (@robertdsimmons) 
   <a href="https://twitter.com/robertdsimmons/statuses/323869157642883074">April 15, 2013</a>
</blockquote>

Sadly, this is only funny because every programmer has inevitably had a conversation that went something like this:

> **Boss:** "We need to make sure these entries are unique."
> 
> **Dev:** &nbsp;"No problem, we can use a GUID."
> 
> **Boss:** "But we are going to be creating millions and millions of these things. What happens if there is a collison?"
> 
> **Dev:** &nbsp;"I don't think it will be a problem."
> 
> **Boss:** "But these are going to be used on the web. What if somebody guesses a valid GUID?"
> 
> **Dev:** "They won't be able to guess, and besides, we will be checking security anyway."
> 
> **Boss:** "I'm just not convinced, I think we should..."At this point, the boss (or other offending party) usually comes up with some&nbsp;cockamamie idea that takes way too much time to implement, and is not guaranteed to be unique.

But I recognize that phrases like "mathematically impossible", "2<sup>122</sup>", and "a trillion times more than the number of stars in the uinverse" are abstract and difficult concepts for business minded folks. So, for the non-anylitical types. I'm going to let you in on the joke, and attempt to show you why we programmers just stare at you blankly when you reject mathematically sound advice.

A GUID is like a really, really big number. How big? 2<sup>122</sup>&nbsp;or 5.3×10<sup>36</sup>

So what does that actually mean? Let's pretend for a minute we had every single person on the earth generate a GUID every second for one year.
<span style="font-size: large;">
</span><span style="font-size: large;">7,000,000,000 ppl * 86400 seconds in a day * 365 days in a year</span>

Exactly how many GUIDs is that? 
<span style="font-size: large;">
</span><span style="font-size: large;">**220,752,000,000,000,000**</span>

Ok, that's pretty impressive, but what percentage of the total number of possible GUIDs is that?

<span style="font-size: large;">**~0.000000000000000000042%**</span>

That doesn't look too good on a&nbsp;PowerPoint&nbsp;slide though, so how long would it take us to get to just 1% of all possible GUIDs?

**<span style="font-size: large;">2,400,884,200,000,000,000 Years</span>**

Great news! It will only take us a measly 2.4&nbsp;Quintilian years to completely exhaust&nbsp;1% of all possible GUIDs if every person on earth spends every second of every day generating them!

If you are a developer and you are faced with this conversation in the future, then just pull up this article and show it to the mathematically challenged.
