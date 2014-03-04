---
layout: post
title: Angular Powered Text Transformations - Part 1
tags: [angular, javascript, ng-transform]
description: Building a text transformation utility using JavaScript and Angular.js
---

###Tell me if this sounds familiar?

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
It might be you need to turn those into JavaScript class properties in one place, and a C# interface
somewhere else.

Now What?

***You brute force it that's what you do!***

###The Problem

You copy and paste and check and double check to make sure you
didn't accidentally screw up in the process. At some point you will probably say to yourself:

> *Oh my goodness if I have to copy and paste this one more time I am going to slit my wrists!*

But then you realize **hey, I'm a programmer, and why do anything more than once** when you can have a computer do it for you.

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
properties that we can bind our controls to, **we need something to model our transformation pipeline.**
To start, we can just use a simple array of functions.

Here is the skeleton:

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

###Transformation Pipeline

Now let's build out that transformation pipeline. We want to keep it as simple as possible for right now, but the
idea is to pass the initial input into the begining of the pipeline, and then pass the output of that into the next
function. Rinse. Repeat.

{% highlight javascript %}
TransformCtrl.prototype.transform = function () {
  //Make copies for posterity
  var input = angular.copy(this.input),
      transformFunctions = this.transformations.slice(0);

  //Kick off the pipeline and keep pushing the output
  // from the last step into the next
  for (var i = 0; i < transformFunctions.length; i++) {
    input = transformFunctions[i].call(this, angular.copy(input));
  }

  this.output = input;
};
{% endhighlight %}

So that's it. Pretty simple to start.

You'll notice that I am making a copy of the input initially to pass into the first function, and then **capturing
the result of each step of the transformation**, which I then copy and pass into the next step. This gives us a nice linear
pipeline, and the foundation to build off of.

###Adding Transformations

Without actually adding some transformations to the mix, this thing is going to be pretty useless, so let's **put together
a few examples to demonstrate the process.**

Suppose we wanted to take our tab separated list of names up there, and turn it into a JavaScript object. We might want
to break that down into roughly 4 steps.

1. Split the single line into an array of strings
2. Alphabetize the names
3. Lower case the first letter
3. Format the remaining output as a simple JS Object

**Each of those steps can be modelled as a function within our pipeline.** So let's go ahead and build them out now.

{% highlight javascript %}

TransformCtrl.prototype.splitWhiteSpace = function (input) {
  return input.split(/\s+/);
};

TransformCtrl.prototype.alphabetize = function (input) {
  input.sort(function (a, b) {
    return a.localeCompare(b);
  });
  return input;
};

TransformCtrl.prototype.lowercaseFirst = function (input) {
  var output = [],
      i = 0;

  for (i = 0; i < input.length; i++) {
    output.push(this.lowercaseFirstSingle(input[i]));
  }

  return output;
};

TransformCtrl.prototype.lowercaseFirstSingle = function (str) {
  return str.replace(/^([A-Z]{1})/g, function (match) {
    return match.toLowerCase();
  });
};

TransformCtrl.prototype.propertiesToJSObject = function (input) {
  var objLines = ["var NewObj = function(config){"];

  //Format each property as "this.name = config.name;"
  angular.forEach(input, function (prop) {
    objLines.push("  this." + prop + " = config." + prop + ";");
  });

  objLines.push("};");

  return objLines.join("\n");
};

{% endhighlight %}

Once we have all of those function added to our controller, we can simply **build up our transformation pipeline by
adding them to the array of functions.** Sprinkle in a little angular binding into our markup and we will have a very
basic text transformation utility.

{% highlight javascript %}

var TransformCtrl = function () {
  this.input = "";
  this.output = "";

  this.transformations = [
    this.splitWhiteSpace,
    this.alphabetize,
    this.lowercaseFirst,
    this.propertiesToJSObject
  ];
};

{% endhighlight %}

###Results

The end result is that you end up transforming this line of text:

{% highlight powershell %}
Name    FaceShape   LegacyProperty  NeverGetsUsed   NobodyKnowsWhatThisMeans KillMeNow
{% endhighlight %}

Into this little code snippet:

{% highlight javascript %}
var NewObj = function(config){
  this.faceShape = config.faceShape;
  this.killMeNow = config.killMeNow;
  this.legacyProperty = config.legacyProperty;
  this.name = config.name;
  this.neverGetsUsed = config.neverGetsUsed;
  this.nobodyKnowsWhatThisMeans = config.nobodyKnowsWhatThisMeans;
};
{% endhighlight %}

{% include livedemo.html url="http://jsfiddle.net/jwcarroll/zwrzf/" %}

**In part 2 we will look at using a service to DRY up our code, add some error handling, and support for multiple transformations.**