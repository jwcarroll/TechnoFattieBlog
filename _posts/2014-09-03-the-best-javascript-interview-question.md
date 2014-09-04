---
layout: post
title: The Best JavaScript Interview Question
tags: [javascript, interviews]
description: >
  If you could ask a single question to a candidate to quickly assess their knowledge of JavaScript, what would
  it be? What if there were a 10 line snippet of code you could use to immediately determine if a candidate was a
  beginner, intermediate, or advanced JavaScript practitioner?
scripts:
  - /js/bootstrap.min.js
---

###Everybody is *doing* JavaScript these days

But considering the barrier to entry is so low, it can be enormously difficult to gauge a persons skill by simply asking them.
**This is a particularly difficult problem when conducting an interview.**

**For those whose native language is not JavaScript**, they might say they really know it well, when in fact they
only know a little bit of jQuery. **For those who have only ever done JS**, they might rate themselves much lower than the
former crowd, when in fact they could code circles around them in JavaScript.

###What you need is a barometer

A way to very quickly determine just how in depth a candidates knowledge of JavaScript goes.

***Does such a magical question exist?***

Well... there is obviously no silver bullet, but I think this little 10 line snippet comes pretty close. Look it over,
and I'll explain why I think it works.

>#### What does the following code do? And why?

```js
falseStr = "false";

if(true){
  var falseStr;

  if(falseStr){
   console.log("false" == true);
   console.log("false" == false);
  }
}
```

###No Peeking!

*Do you know the answer?*

*Do you know why?*

<div class="panel-group" id="accordion">
  <div class="panel panel-danger">
    <div class="panel-heading">
      <h4 class="panel-title">
        <a data-toggle="collapse" data-parent="#accordion" href="#collapseOne">
          Click Here to Reveal The Answer...
        </a>
      </h4>
    </div>
    <div id="collapseOne" class="panel-collapse collapse">
      <div class="panel-body">
        <p>When the above script is executed it will actually print out the following:</p>
        <kbd>false</kbd>
        <br />
        <kbd>false</kbd>
      </div>
    </div>
  </div>
</div>

###JavaScript is weird man

If you didn't already know the answer did it surprise you?

This is a deceptively simple question, but there are **four very fundamental concepts** being demonstrated in this tiny example.

###Hoisting and Scoping

**[Hoisting is one of the more esoteric and tricky aspects of JavaScript][hoisting]**, but important to understand nonetheless. It dictates
how your code will actually be interpreted by the compiler and executed. Variable, and function definitions will all get
pulled up to the top of their respective scope ***before their assignments.***

This means that the above code will actually be interpreted like this:

```js
var falseStr;
falseStr = "false";

if(true){
  //var moved to top
  ...
}
```

Without that tidbit of knowledge you might be inclined to think that <code>falseStr</code> starts off as a global variable
and then gets redefined within the first if block.

The second concept important to understanding this is that **[JavaScript doesn't have block level scope, it has function
level scope][scope].** So despite the <code>var falseStr;</code> appearing inside an if block, it is scoped outside of that
block to either the containing function, or in this case, the global scope.

###Type Coercion: Boolean Expressions

**Type Coercion is a feature of JavaScript**, and one that people talk about in almost mystical terms... as though the rules
for boolean expressions and comparisons were not well known, but rather mysterious and subject to the whims of the runtime
on any particular day.

The truth is (no pun intended) that **the rules are well known, and [documented for anybody who cares to read them][boolean-expression].**

A boolean expression is coerced into a boolean value using a system level function that has explicit rules for what will
evaluate to true and false. This is what is called **truthy** and **falsey** values in JavaScript.

```js
//truthy values
var truthyValues = [true, 1, "nonEmptyString", [], {name:'Just an object'}];

//falsey values
var falseyValues = [false, 0, "", null, undefined];
```

If we were to explicitly convert our variable to a Boolean, you would see that **the non-empty string converts to true.**

```js
var falseStr = "false";

if(Boolean(falseStr)){
  ...
}
```

###Type Coercion: Loose Comparison

**However, when doing a comparison using double equals (<code>==</code>) then the rules are slightly different.** If you want
to know all the rules you can [read about them here][loose-comparison], but for our purposes you only need to know that when JavaScript does
a loose comparison on a <code>Boolean</code> and a <code>String</code>, it attempts to convert them both to a <code>Number</code>
before doing a strict comparison.

If we rewrite the code snippet to be explicit, then **it makes complete sense why both comparisons fail.**

```js
console.log(Number("false") == Number(true));
//console.log(NaN == 1);

console.log(Number("false") == Number(false));
//console.log(NaN == 0);
```

Clearly NaN does not equal 1, and NaN also does not equal 0.

###Just Use === You Idiot!

It's true that always using strict comparison would allow you to avoid the aforementioned headaches, but this is about
why. **The reason Douglas Crockford put this rule into JSLint is because he understands the rules.**

If you can't explain why you always use strict equality comparison then you are guilty of [cargo cult programming][cargo-cult].

###You are an @#%*&*!

At this point you may be thinking I'm just a big jerk who likes to throw tricky questions at candidates just to make them
feel bad and make myself look smart.

That's a fair criticism, but unfounded. **I would never use a question like this to base a hiring decision off of**, but if a
candidate tells me they are an 8 in JavaScript... then they should be able to nail this question. **It's meant to be a quick
calibration of depth of knowledge in the JavaScript language**. Nothing more. Nothing less.

I would also never ask this question without first explaining what the intent was. **It's meant to be a starting point for
a conversation, not a pass/fail gatekeeper.**

###Conclusion

I like it because although the code is tricky, **it's still simple enough that you don't have to stare at it for twenty
minutes** trying to figure out what it does.

**I like it because it covers some core language features** that are important for anybody doing heavy JavaScript programming.
It's not exhaustive, but it is deep enough that if you nail it, there is a good chance you have taken the time to really
study the language.

You may think questions like this are gimmicky, but I can guarantee if you are interviewing someone and they nail this
question, then **they have done more than just written a few lines of jQuery.**

[hoisting]: http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html
[scope]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions
[boolean-expression]: http://www.ecma-international.org/ecma-262/5.1/#sec-12.5
[loose-comparison]: http://www.ecma-international.org/ecma-262/5.1/#sec-11.9.3
[cargo-cult]: http://en.wikipedia.org/wiki/Cargo_cult_programming