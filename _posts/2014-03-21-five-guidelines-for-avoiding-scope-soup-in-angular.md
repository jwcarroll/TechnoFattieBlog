---
layout: post
title: 5 Guidelines For Avoiding Scope Soup in Angular
tags: [angular, javascript, stackoverflow]
description: "Scope Soup is when you build a tangled mess of Angular code that is completely coupled to $scope in really terrible ways.
              The unfortunate part is that you see a lot of examples like that. I've written code like that. But after a year and a half
              working on a massive Angular project, I can assure you there is a better way."
---

###Today I was answering a question on Stackoverflow about Angular.js

The more I dug into this person's particular problem the more help I realized they needed. They were admittedly an
Angular noob, but making a pretty decent effort by getting their hands dirty. A+ for effort.

I ended up completely refactoring their code, but **[I see these same kinds of problems][3] over and over again so I decided
to compile some of these guidelines into a blog post for future reference.**

If you are impatient, here is the [original code](http://plnkr.co/edit/0aYg1TbnRFwW1WhuPKYp?p=preview),
and here is the [refactored version](http://plnkr.co/edit/6R3zEX?p=preview).

The code is full of what I call <span class="keyword">scope soup</span>. **What is Scope Soup you ask?**

This is...

{% highlight javascript %}
angular.module('app', [])
.controller('MyCtrl', function($scope, $http){
  $scope.tagSearch = '';
  $scope.tags = [];

  $scope.getTags = function(){
    $http.get('/api/tags/?search=' + $scope.tagSearch)
    .success(function(data){
      $scope.tags = data;
    });
  };

  $scope.$watch('tagSearch', function(){
    $scope.getTags();
  });

  // 1000 more lines of this mess

});
{% endhighlight %}

AAAAHHHHHH!!!! Somebody save me from the madness!

**<span class="keyword">Scope Soup</span> is when you build a tangled mess of Angular code that is completely
coupled to `$scope` in really terrible ways.** The unfortunate part is that you see a lot of examples like that.
I've written code like that. But after a year and a half working on a [massive Angular project][3],
I can assure you there is a better way.

**So without further ado, here are my <span class="keyword">five guidelines for avoiding Scope Soup in Angular.</span>**

###1. Controllers Should Be Classes

Angular makes it really easy to add an inline controller as an anonymous function, **but just because you can doesn't
mean you should.** Inline functions encourage you to shove everything into `$scope`, but defining your controller as a class
and using the prototype to add functionality helps to keep you honest.

** **Update** **
*A colleage of mine, correctly pointed out that the `this` pointer was refering to `$scope` instead of the controller.
I forgot to pull out a reference to the controller using a closure. You could also do this using the `.bind()` method.*

{% highlight javascript %}
//Don't do this
app.controller('MyCtrl', function($scope){
  $scope.doStuff = function(){
    //Really long function body
  };
});


//Do this instead
var MyCtrl = function($scope){
  var _this = this;

  $scope.doStuff = function(){
    _this.doStuff();
  };
};

MyCtrl.prototype.doStuff = function(){
  //Really long function body
};

MyCtrl.$inject = ['$scope'];

app.controller('MyCtrl', MyCtrl);
{% endhighlight %}

Defining your controllers in this way makes you think a little harder about just hanging everything off of `$scope`. Aside
from that, if your controller has more than a couple of methods, tracing the code through well defined methods is a lot
easier than trying to sift through a mess of <span class="keyword">Scope Soup</span>.

###2. Keep Controllers Skinny With Services

Rest services are usually pretty simple, so it can be tempting to just use `$http` to get what you need right in
your controller. However, **it's much better to encapsulate those calls into a service.**

{% highlight javascript %}
//Don't do this
// ng-click='getDetail()'
$scope.getDetail = function(){
  $http.get('/api/item/detail/' + $scope.selectedItem.id)
  .success(function(detail){
    $scope.itemDetail = detail;
  });
};

//Do this instead
// ng-click='getDetail(item)'
MyCtrl.prototype.getDetail = function(item){
  this.itemService.getDetail(item)
  .success(function(detail){
    item.detail = detail;
  });
};
{% endhighlight %}

Again, doing this will make your code easier to understand. Not only that but it will be much easier to test
since you can mock the service and inject it into your controller. **It helps you to avoid <span class="keyword">Scope Soup</span>
because you have to decouple your data services from your controller.**

Pulling out data access into a service will also allow you to reuse that same code within
directives, filters, other controllers, and other services.

###3. Eliminate Scope Using The 'Controller As' Syntax

Angular 1.2 introduced the ability to publish your entire controller to `$scope` using the ["Controller As"](http://docs.angularjs.org/api/ng/directive/ngController)
syntax. Using this you can **completely eliminate the need to reference `$scope` in your controller.**

The syntax is pretty simple as well.

{% highlight html %}
<!-- In Your Binding -->
<div ng-controller="MyCtrl as ctrl">
   <span>{{ctrl.name}}</span>
</div>

//In your route
$routeProvider
.when('/',{
  templateUrl: 'foo.html',
  controller: 'MyCtrl',
  controllerAs: 'ctrl'
});
{% endhighlight %}

A controller should be pretty self contained, and **this technique will also keep you honest and less likely to
rely on properties being set in the parent scope** which can lead to difficult to maintain code, and hard to track down bugs.

{% highlight javascript %}
//Don't do this
app.controller('MyCtrl', function($scope){
  $scope.name = 'Techno Fattie';
});

//Do this
var MyCtrl = function(){
  this.name = 'Techno Fattie';
};

app.controller('MyCtrl', MyCtrl);
{% endhighlight %}

###4. Eliminate Watches Using 'ng-change' And ES5 Properties

Getting rid of `$scope` completely might seem impossible if you are using explicit watches in your controllers, but in
most cases you shouldn't need them.

**If you have a watch set up to listen for a property change that originates from a form field, then [ng-change](http://docs.angularjs.org/api/ng/directive/ngChange)
is your best bet.** It's important to note however, that ng-change requires ng-model, and does not fire a change event if the change
originates from the controller. That being said, this is a pretty common scenario where I see people use a `$watch` instead
of just using ng-change.

{% highlight html %}
//Don't do this
<input type="text" ng-model="name" />

app.controller('MyCtrl', function($scope){
  $scope.$watch('name', function(){
    //call some service here
  });
});

//Do this
<input type="text" ng-model="ctrl.name" ng-change="ctrl.search(ctrl.name)" />

MyCtrl.prototype.search = function(name){
  //call some service here
};
{% endhighlight %}

If you have some property that isn't bound to an input field, or is going to be updated from code, it may seem like
a watch is your only choice. However, if you don't have to support IE8 or lower, then **you can take advantage of [ES5
properties][2] to trigger functionality when something changes on your controller.**

{% highlight javascript %}
//Don't do this
app.controller('MyCtrl', function($scope){
  $scope.$watch('selectedItem', function(){
    //call some service here
  });
});

//Do this instead
var MyCtrl = function(){
  this._selectedItem = null;
};

Object.defineProperty(MyCtrl.prototype,
    "selectedItem", {
    get: function () {
        return this._selectedItem;
    },
    set: function (newValue) {
        this._selectedItem = newValue;

        //Call method on update
        this.onSelectedItemChange(this._selectedItem);
    },
    enumerable: true,
    configurable: true
});
{% endhighlight %}

This might seem a bit verbose, but it does help you avoid <span class="keyword">Scope Soup</span> by eliminating your
need to use a `$watch`. Your controller is a little easier to test with one less dependency and it's a lot more reusable.

###5. Don't Use Scope To Pass Data Around

Scope can be a convenient way to pass data between two controllers, it might even feel a little ninja, but **it will
almost always end in tears and heavy drinking.**

The problem with relying on values to be set from a parent controller, or relying on a child controller to set data
back up on the parent scope, is that you **end up with an implicit coupling that isn't easy to see at first.** If I'm looking
at my controller and see some random property being used, but it's never set, and doesn't exist in my view, then I have to
go hunting around to find out where it came from.

Worse, it means **the controller implementation is completely dependent on the ordering of the bindings in the view!**
I can't use or test those controllers independently of each other, because they are coupled together by `$scope`.

{% highlight javascript %}
//Don't do this
app.controller('Parent', function($scope){
  var sharedObj = {
    someProperty: 'foo'
  };

  $scope.sharedObj = sharedObj;

  $scope.$watch('sharedObj.someProperty', function(){
    //call some service here
  });
});

app.controller("Child", function($scope){
  $scope.$watch('sharedObj.name', function(){
      $scope.myName = $scope.sharedObj.name;
  });
});


//Do this instead
var Parent = function(sharedDataService){
  sharedDataService.updateName('foo');
};

var Child = function(messageService){
  sharedDataService.onNameChanged(this.nameChangedHandler);
};

Child.prototype.nameChangedHandler = function(){
  this.myName = sharedDataService.getName();
};
{% endhighlight %}

Using a messaging paradigm to communicate between controllers will help you keep things clean, decoupled, and testable.
**It helps you to avoid <span class="keyword">Scope Soup</span> because you aren't using `$scope` like a peeping tom...
taking advantage of your position and knowledge in order to do something you know you probably shouldn't.**

Some of you might be objecting right now that I could have done the same thing by using `$emit` and `$broadcast` to accomplish
the same thing, but **I prefer to keep those details hidden for the same reason I keep `$http` hidden behind a service layer.**
It also means I am not dependent on having a particular parent-child relationship configured in order to make this work.
Another advantage is that my service could be doing a lot more stuff under the hood if needed. It could be calling a
web service, or pushing out a message using Web Sockets, but the code in my controllers won't have to change.

###Conclusion

Angular's Scope is the magic glue that makes everything work together, but it is easily abused. It is better kept behind
the scenes doing the hard work of keeping your UI in sync with your underlying model. With a little bit of work, you can
completely eliminate the need to litter your code with `$scope`.

**Stop Making Scope Soup!**

In the end you will have a much more testable, and maintainable application.

  [1]: http://plnkr.co/edit/vLE3vwFTlEaIjQyvbULt?p=preview
  [2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
  [3]: http://wintellect.com/html5-javascript-experts-angular-consultants