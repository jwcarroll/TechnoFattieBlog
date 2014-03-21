---
layout: post
title: Angular Powered Text Transformations - Part 2
tags: [angular, javascript, ng-transform]
description: Building a text transformation utility using JavaScript and Angular.js. Using services to clean up our code.
---

In the last installment of this series, we created a simple text transformation utility with Angular.js

It allowed us to create a simple pipeline for transforming some input text into something else using JavaScript. It
isn't terribly useful right now, so we are going to use Angular's built in goodness to DRY up our code a little and
make it more useful.

[You can check out the demo of the original here.](http://jsfiddle.net/jwcarroll/zwrzf/ "Simple Angular Text Transformation Utility")

###Leveraging Angular Services

In Part 1, everything was contained in a single controller. Which is ok for a demo, but terrible for well... pretty much
anything else. It's a good idea to keep your controllers skinny, so let's put this one on a diet.

The first thing to do is refactor the actual transformation pipeline out into a service.

{% highlight javascript %}

var TextTransformation = function(){
  this.pipeline = [];
};

TextTransformation.prototype.addTransformation = function(transform){
  this.pipeline.push(transform);
};

TextTransformation.prototype.transform = function(input){
  var inputCopy = angular.copy(input),
    transformFunctions = this.pipeline.slice(0);

  for (var i = 0; i < transformFunctions.length; i++) {
    inputCopy = transformFunctions[i].call(this, angular.copy(inputCopy));
  }

  return inputCopy;
};

{% endhighlight %}

Nothing groundbreaking here, we just pulled the responsibility for running the pipeline out into a service, which we
can then use inside our controller. It's already a little more flexible than the monolithic controller because we could
run the same transformation on multiple different inputs if we wanted.

Now we can refactor our controller a bit to use the new service. Assuming we have the same basic functions as last time
our new controller should look something like this:

{% highlight javascript %}

var TransformCtrl = function (textTransformation) {
  //Sample input
  this.input = "Name    FaceShape   LegacyProperty  NeverGetsUsed   NobodyKnowsWhatThisMeans KillMeNow";
  this.output = "";

  this.textTransformation = textTransformation;

  this.textTransformation.addTransformation(this.splitWhiteSpace);
  this.textTransformation.addTransformation(this.alphabetize);
  this.textTransformation.addTransformation(this.lowercaseFirst);
  this.textTransformation.addTransformation(this.propertiesToJSObject);
};

TransformCtrl.prototype.transform = function () {
  this.output = this.textTransformation.transform(this.input);
};

TransformCtrl.$inject = ['textTransformation'];

{% endhighlight %}

Sweet! Now our controller and transformation pipeline are independent which makes them easier to test and reuse.

### Handling Errors

So it's great that we did a little code cleanup, but this utility is still pretty limited and doesn't even have a
mechanism for basic error handling. Any single transformation in the pipeline could fail and we would never know about it.
That's not exactly the best user experience, so let's make that better.

The easiest thing to do is just wrap the whole process in a try catch, and then format the error as a string, but a
better approach would be to return an object indicating success/failure and letting the consumer decide what to do from there.



**In part 2 we will look at using a service to DRY up our code, add some error handling, and support for multiple transformations.**