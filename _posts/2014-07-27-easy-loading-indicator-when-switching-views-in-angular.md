---
layout: post
title: Easy Loading Indicator When Switching Views In Angular
tags: [angular, javascript, directives, routing]
description: Sometimes a route change in Angular can take a little while, but using this simple trick you can show a loading indicator while waiting on your view to load.
---

It's no doubt **Angular is an awesome framework for building modern web applications.** Loading tiny bits of HTML and JSON
can be amazingly fast, and SPA's are a great way to give your users a snappy UI without the overbearing
page refresh of traditional server-side rendering.

**However...**

Occasionally you need to load more than a little data, and that can present some unfortunate usability issues in
your application. The good news is that **Angular gives us all the tools we need to solve these issues easily**, but first
let's look at some common problems.

##Hurry Up And Wait!

It's pretty reasonable for a controller to do some initialization to load data from the server, but sometimes this can
result in what I like to call the **Fast Route, Slow Controller** problem. This is where the primary data needed for the
view loads slowly, and the user is left staring at an uninitialized view, wondering if something went wrong.

Imagine we have a simple controller that loads some data and displays it in a table:

```javascript
var SlowCtrl = function(slowDataService){
  this.slowDataService = slowDataService;
  this.init();
};

angular.extend(SlowCtrl.prototype, {
  contacts:[],
  init:function(){
    var _this = this;

    this.slowDataService.getContacts()
      .then(function(contacts){
        _this.contacts = contacts;
      });
  }
});
```

Using this controller in one of our routes would result in a fast route change, but a **there will be a long, uncomfortable
lag time waiting for the data to load.**

{% include embed-plunker.html plunk_id="p1xfXHk6ln0iWbx6cmbo" %}

As you can see, the view loads up pretty quickly, but then we are stuck waiting a long time for the data to load.

You may already be thinking that **using the `resolve` property on a route is a good way to solve this issue** by letting
Angular load all the data before showing the view. This certainly puts us on the right track, but leads to our second
most common problem, **Slow Route, Fast Controller**.

##Is Something Supposed To Be Happening?

If we move our slow call to a resolve in our route definition like this:

```javascript
.when('/route1', {
      templateUrl:'route1.html',
      controller:'slowCtrl',
      controllerAs:'ctrl'
      resolve: {
        //Don't do this in a real app, instead
        // use an object that encapsulates this
        contacts: ['slowDataService', function(slowDataService){
          return slowDataService.getContacts();
        }]
      }
    })

//Now controller has access to contacts on load
var SlowCtrl = function(contacts){
  this.contacts = contacts;
};
```

Then we can ensure the data is loaded before our controller is instantiated, and the view loaded into the DOM. The issue
is that **now the painful lag has simply been moved to our route changing**, which leaves users wondering if they clicked
the button or not.

{% include embed-plunker.html plunk_id="uFey5hcPnymF9BF053qC" %}

##Directives To The Rescue!

The good news is we can fix this with a simple directive. **Angular has a number of built in events that is publishes
using the `$scope` service that you can register handlers with.** It just so happens that the built in `$routeProvider`
publishes three different events during routing.

[$routeChangeStart](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeStart)

[$routeChangeSuccess](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeSuccess)

[$routeChangeError](https://docs.angularjs.org/api/ngRoute/service/$route#$routeChangeError)

By tapping into these events, **we can use a directive to hide/show some kind of loading indicator** while waiting on our
views to load.

<div class="alert alert-danger" role="alert">
<strong>Warning:</strong> normally referencing <code>$rootScope</code> is an indicator you are doing something wrong, but in this case we need to be able
to handle these events at the highest level.
</div>

```javascript
var routeLoadingIndicator = function($rootScope){
  return {
    restrict:'E',
    template:"<h1 ng-if='isRouteLoading'>Loading...</h1>",
    link:function(scope, elem, attrs){
      scope.isRouteLoading = false;

      $rootScope.$on('$routeChangeStart', function(){
        scope.isRouteLoading = true;
      });

      $rootScope.$on('$routeChangeSuccess', function(){
        scope.isRouteLoading = false;
      });
    }
  };
};
```

Now when we can simply drop this directive into our markup somewhere outside our `ng-view` and we can **give the user
immediate feedback that something is happening by showing the loading indicator.** Also, since we publish the state out
to scope, we can use that to hide any existing view while we wait for the new one to load.

```html
<route-loading-indicator />

<div ng-if='!isRouteLoading' ng-view></div>
```

{% include embed-plunker.html plunk_id="TNwjw5hlSc2eYROVV10E" %}

##Conclusion

**Long load times in between AJAX requests can lead to an awkward user experience**, leaving the user unsure if the application
is actually doing anything. Angular makes it easy to handle these scenarios using directives.

**Using these techniques you can customize the loading experience to suit your needs.** Here is an example using
[spin.js](http://fgnass.github.io/spin.js/) and a few animations to create an interesting transition.

{% include livedemo.html url="/demos/angular-route-loading-indicator" %}