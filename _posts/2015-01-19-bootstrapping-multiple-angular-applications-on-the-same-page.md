---
layout: post
title: Bootstrapping Multiple Angular Applications Automatically Using 'multi-app'
description: If you have ever wanted to run multiple Angular applications in the same page you will understand it
             simply isn't possible with ng-app. However, there is another way, and this article explains how to do it.
tags: [angular, javascript, directives, angular-modules, bootstrapping]
---

###I Have Two Angular Applications... One For Each of You!

AngularJS is amazingly flexible, so it usually comes as a surprise to people when they find out that **you can't use
`ng-app` twice on the same page.** Granted, this isn't something you *really* need too much, but most of us
have thought of a reason or two that it might be handy.

Now, it's not like this is impossible, but it does require manually bootstrapping the application using
[angular.bootstrap](https://docs.angularjs.org/api/ng/function/angular.bootstrap).

```javascript
var myTargetElement = document.getElementById('my-target');

angular.bootstrap(myTargetElement, ['some-module']);
```

That's not *terrible*, but [on my previous blog post about a simple loading indicator]({% post_url 2015-01-09-another-angular-loading-indicator %}),
I really **needed an easy way to automatically make this happen**. Ideally it would work just like `ng-app` did, so I
threw together some code to do this for me.

Turns out, it was ridiculously simple:

```javascript
//Find all elements with an attribute of 'multi-app'
var allMultiElements = document.querySelectorAll('*[multi-app]');

//Bootstrap each element using the value of the attribute
// as the module name
angular.forEach(allMultiElements, function(elem){
    var appName = angular.element(elem).attr('multi-app');

    angular.bootstrap(elem, [appName]);
});
```

###TADA!

I use the native [`querySelectorAll`](https://developer.mozilla.org/en-US/docs/Web/API/Document.querySelectorAll) to grab
all elements that have an attribute of `multi-app`. Then, since I already know I am using AngularJS, I can simply lean
on it's built in utility function `forEach` to loop over those elements and bootstrap each one using the attribute value
as the module name.

In order to put a bow on this, and make the whole thing automatic, **I wrap the above code into a function, and then call
it whenever the document is ready.**

```javascript
(function () {
    'use strict';

    angular.element(document)
        .ready(bootstrapAllModules);

    function bootstrapAllModules(){
        var allMultiElements = document.querySelectorAll('*[multi-app]');

        angular.forEach(allMultiElements, function(elem){
            var appName = angular.element(elem).attr('multi-app');

            angular.bootstrap(elem, [appName]);
        });
    }
}());
```

Now you can automatically bootstrap multiple applications into the DOM like this:

```html
<div multi-app="app-one"></div>
<div multi-app="app-two"></div>
<div multi-app="app-three"></div>
```

And... the entire thing is **only 12 lines of code!**

For those of you who are too lazy to <kbd>CTRL + C, CTRL + V</kbd> I put [the whole thing up on GitHub](https://github.com/jwcarroll/angular-multi-app).

There is **even a bower package**, which is hilariously comical to me, but nonetheless you are free to:

<kbd>bower install angular-multi-app</kbd>

Have fun, and don't shoot yourself in the eye with this thing!