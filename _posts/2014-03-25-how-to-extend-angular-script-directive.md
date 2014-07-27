---
layout: post
title: How To Extend Angular's Script Directive
tags: [angular, javascript, directives]
description: "Angular's built in script directive is great, but limited. However, we can extend the built in directive
             by taking advantage of decorators, and a little known feature in Angular."
---

**Angular's built in [script directive][1] is really nice for embedded templates, but not much else.** If you are like me, you
have found yourself wanting to be able to extend it to work for other types of embedded data. Luckily this turns out
to be pretty easy, but not obvious.

###The Problem

Angular allows you to specify templates inline using the `<script>` tag, but it only supports a single *type* of
`type="text/ng-template"`.

**But what if you've got a view that also has some context specific info that you want to make available to the client.**
You don't want to make *another* trip to the server to get it, so you encode it as JSON, and embed it into a `<script>` tag.

```html
<script type="text/context-info">
{
   "name":"foo-view",
   "id":34,
   "tags":[
      "angular",
      "javascript",
      "directives"
   ]
}
</script>
```

**Wouldn't it be nice if you could just teach the script directive to handle a new type?**

Oh but you *can!*

###Decorating is Fun!

One amazing feature of Angular is the ability to **intercept a service at the time of creation** and do something with it.
While you may already know about the [`$provide.decorator`][2] method, you may not know that you can also **intercept a directive.**

What does that look like?

```javascript
angular.module('myApp').config([
    '$provide',
    function ($provide) {
        $provide.decorator('scriptDirective', function($delegate){
          //$delegate is an array, and index '0' is the Angular directive
        });
    }
]);
```

That will let us grab the directive before it ever gets used, and extend it as we please. **We can use the amazing flexibility
of JavaScript to add new functionality to the existing directive in order to achieve our desired results.**

The script directive uses the `compile` method to work it's magic so that is where we are going to tap in. The first thing
we need to do is grab a reference to the original compile method.

```javascript
var scriptDirective = $delegate[0],
    originalCompile = scriptDirective.compile;
```

**Now we can replace the compile method with our own version** that will check the type attribute for our custom type
and handle it accordingly. In order to make sure we don't mess up the existing directive we simply let it fall through
to the original compile method.

Finally we just return the intercepted `$delegate` once the modifications have been made.

```javascript
scriptDirective.compile = function(elem, attr, transclude){
  if(attr.type === 'text/context-info'){
    var contextInfo = JSON.parse(element[0].text);

    //Custom service that can be injected into
    // the decorator
    contextInfoService.addContextInfo(contextInfo);
  }

  originalCompile(elem, attr, transclude);
};

return $delegate;
```

{% include livedemo.html url="http://embed.plnkr.co/mSFgaO/preview" %}

**Voila! Now you can extend the built in script directive with your own custom types.**

###Conclusion

Using the built in `$provide` service and the dynamic nature of JavaScript, we were able to extend the built in
script directive. **Angular is amazingly flexible. If it seems like there is a limitation, chances are there
is an easy hook available to get the functionality you want without a lot of fuss.**

[1]:http://docs.angularjs.org/api/ng/directive/script
[2]:http://docs.angularjs.org/api/auto/object/$provide#decorator