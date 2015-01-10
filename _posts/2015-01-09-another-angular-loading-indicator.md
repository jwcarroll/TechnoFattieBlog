---
layout: post
title: Another Angular Loading Indicator Directive
description: Have you ever needed a simple and flexible loading indicator in Angular? This post walks you through how to build a reusable directive for doing just that.
tags: [angular, javascript, directives]
scripts:
  - /js/angular/angular.min.js
  - /js/another-angular-loading-indicator/samples.js
---

###How Do I Show A Loading Indicator In Angular?

It's a question I run into a lot actually.

To be fair, there are a million different types of loading indicators and ways to use them,
but most often, **developers are looking for a simple solution**. They need a little spinner
next to a label... or inside a button.

Again, nothing fancy, but **it does need to be flexible.**

In it's simplest form, a loading indicator can be accomplished in Angular with a combination of
`ng-show` and a simple boolean field on your controller.

Using Bootstrap and Font Awesome classes the markup might look something like this:

```html
<!-- simple spinning cog icon -->
<i class="fa fa-cog fa-spin ng-hide" ng-show="ctrl.isBusy"></i>
```

And here is an example of **using it inside a button**

<div class="example-code" data-example-name="simple-boolean-example" ng-controller="simpleBooleanController as ctrl">
  <button type="button"
    class="btn btn-lg btn-primary"
    ng-click="ctrl.makeCall(2000)"
    ng-disabled="ctrl.isBusy">Slow Call <i class="fa fa-cog fa-spin ng-hide" ng-show="ctrl.isBusy"></i>
  </button>
  <span class="h2" ng-bind="ctrl.result | json"></span>
</div>

###Making It Modular

This isn't exactly earth shattering code I know, but **if you pepper this little snippet of code all
over the place, it's going to suck to make changes in the future**. This is a great place to introduce a
directive to encapsulate our little spinner.

```javascript
function Spinner(){
  return {
    restrict: 'E',
    template: '<i class="fa fa-cog fa-spin ng-hide" ng-show="show"></i>',
    scope: {
      show: '='
    }
  };
}

angular.module('myModule')
  .directive('spinner', Spinner);
```

That's about as simple as we can possible make this thing. It doesn't even need a link function because we
are using `isolate scope` to pull in our property to use with `ng-show`.

**Using this in code becomes pretty trivial:**

```html
<!-- so easy! -->
<spinner ng-show="ctrl.isBusy" />
```

And of course here is our live example using the directive this time... *view source if you don't believe me.*

<div class="example-code" data-example-name="simple-spinner-directive-example">
  <button type="button"
    class="btn btn-lg btn-primary"
    ng-click="isBusy = !isBusy">Toggle Spinner
  </button>
  <span class="h2"><spinner show="isBusy" /></span>
</div>

###Making It Useful

This simple little spinner might be good for a demo, but **what happens if your call is really fast? Say less than 200ms?**

See for yourself...

<div class="example-code" data-example-name="fast-spinner-example" ng-controller="simpleBooleanController as ctrl">
  <button type="button"
    class="btn btn-lg btn-primary"
    ng-click="ctrl.makeCall(100)"
    ng-disabled="ctrl.isBusy">Fast Call <spinner show="ctrl.isBusy" />
  </button>
  <span class="h2" ng-bind="ctrl.result | json"></span>
</div>

That little blip is pretty annoying. **A real production spinner should take into account some sort of delay**, and preferably
this would be something we could configure right in the directive.

Modifying our existing directive to account for a delay is certainly more code, but also not terribly complicated. All we
need to do is `$watch` for changes to the boolean, and then take advantage of Angular's built in `$timeout` service to
ensure we honor the delay time. If the value flips from `true` to `false` before the timer kicks in, we simply cancel it.

**Putting this all together, here is what our modified spinner directive ends up looking like:**

```javascript
function SpinnerDirective($timeout) {
  return {
    restrict: 'E',
    template: '<i class="fa fa-cog fa-spin"></i>',
    scope: {
      show: '=',
      delay: '@'
    },
    link: function(scope, elem, attrs) {
      var showTimer;

      //This is where all the magic happens!
      // Whenever the scope variable updates we simply
      // show if it evaluates to 'true' and hide if 'false'
      scope.$watch('show', function(newVal){
        newVal ? showSpinner() : hideSpinner();
      });

      function showSpinner() {
        //If showing is already in progress just wait
        if (showTimer) return;

        //Set up a timeout based on our configured delay to show
        // the element (our spinner)
        showTimer = $timeout(showElement.bind(this, true), getDelay());
      }

      function hideSpinner() {
        //This is important. If the timer is in progress
        // we need to cancel it to ensure everything stays
        // in sync.
        if (showTimer) {
          $timeout.cancel(showTimer);
        }

        showTimer = null;

        showElement(false);
      }

      function showElement(show) {
        show ? elem.css({display:''}) : elem.css({display:'none'});
      }

      function getDelay() {
        var delay = parseInt(scope.delay);

        return isNaN(delay) ? 200 : delay;
      }
    }
  };
}
```

And of course, a live example of **using the directive with both fast and slow calls.**

<div class="example-code" data-example-name="final-example" ng-controller="simpleBooleanController as ctrl">
  <button type="button"
    class="btn btn-lg btn-primary"
    ng-click="ctrl.makeCall(2000)"
    ng-disabled="ctrl.isBusy">Slow Call <spinner show="ctrl.isBusy" />
  </button>
  <button type="button"
      class="btn btn-lg btn-danger"
      ng-click="ctrl.makeCall(100)"
      ng-disabled="ctrl.isBusy">Fast Call <spinner show="ctrl.isBusy" />
    </button>
  <span class="h2" ng-bind="ctrl.result | json"></span>
</div>

###Conclusion

As you can see, the spinner only shows up after the default `200ms` and this is something that can be overriden using
the `delay` attribute in the directive.

That's pretty much all there is to it!

Happy coding, and feel free to use this spinner however you want.