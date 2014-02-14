---
layout: post
title: Angular Powered Text Transformations - Part 1
tags: [angular, javascript]
---

**Tell me if this sounds familiar?**

You have some uniform text you just copied from somewhere, like a set of column or property names.
Maybe you got it from a spreadsheet. Maybe you copied it from the results of a query you ran... or maybe
it was buried in the requirements somewhere.

Regardless you've got something that looks like this:

{% highlight powershell %}
Name    FaceShape   LegacyProperty  NeverGetsUsed   NobodyKnowsWhatThisMeans KillMeNow
{% endhighlight %}

Or maybe even this:

{% highlight powershell %}
Name
FaceShape
LegacyProperty
NeverGetsUsed
NobodyKnowsWhatThisMeans
KillMeNow
{% endhighlight %}

**And now you need to use those same names in your code, and probably in more than one place.**
It might be you need to turn those into C# class properties in one place, and a TypeScript interface
somewhere else.

**Now What?**

You brute force it that's what you do!

###The Problem

You copy and paste and check and double check to make sure you
didn't accidentally screw up in the process. At some point you will probably say to yourself:

> Oh my goodness if I have to copy and paste this one more time I am going to slit my wrists!

But then you realize hey, I'm a programmer, and why do anything more than once when you can have a computer do it for you.

Unfortunately being the analytical type, you will also probably formulate a graph in your head that looks something like this:

![Automating vs. Manually Copying And Pasting](/img/automate-vs-manual.png "Automating vs. Manually Copying And Pasting")

Realizing that **the time it would take to automate this, for a one-off solution, wouldn't be any faster than
just doing it manually**, you decide to bite the bullet and get back to it. So you mumble some obscenities under your
breath and go back to copy-pasting your way to hating life.

<h3>Angular To The Rescue! <small>...and javascript</small></h3>

Wouldn't it be nice if there were like a mini-ETL application that let you put together little text transformations
and run them whenever you wanted to? Wouldn't it be nice if it were online so you could get to it from anywhere?

**Angular.js gives us all the tools we need to build such a tool**, so why don't we dive in and get started.

We'll keep this basic for part 1 and slowly evolve it to be more useful. Visually we only need three things:

1. An input for our raw text
2. An output for the result
3. A button to initiate the transform

Using Bootstrap the markup might look something like this:

{% highlight html %}
<div>
  <form role="form">
    <div class="form-group">
      <label for="tbInput">Input</label>
      <textarea id="tbInput" class="form-control" rows="6"></textarea>
    </div>
    <div class="form-group">
      <label for="tbOutput">Output</label>
      <textarea id="tbOutput" class="form-control" rows="6"></textarea>
    </div>
    <div class="form-group">
      <button type="button" class="btn btn-default">Submit</button>
    </div>
  </form>
</div>
{% endhighlight %}

**The goal is to compose our transformation into a series of small steps, where the output
from the last step becomes the input to the next.**

![Transformation Pipeline](/img/transformation-pipeline.png "Transformation Pipeline")

Now let's create a simple Angular module and controller we can use to bind to that markup. Besides having some
properties that we can bind our controls to, we need something to model our transformation pipeline.
To start, we can just use a simple array of functions.

{% highlight javascript %}
var TransformCtrl = function () {
  this.input = "";
  this.output = "";

  //This will eventually hold functions
  this.transformations = [];
};

TransformCtrl.prototype.transform = function () {
  //Handle the transformation pipeline here
};

var module = angular.module('ng-transform', []);

module.controller('transformCtrl', Controllers.TransformCtrl);
{% endhighlight %}

