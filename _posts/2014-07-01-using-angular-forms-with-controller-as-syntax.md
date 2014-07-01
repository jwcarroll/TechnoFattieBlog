---
layout: post
title: Using Angular Forms With Controller As Syntax
tags: [angular, javascript, angular-tips, controller-as]
description: "Accessing Angular's FormController directly off your controller is easy with this little known feature."
---

**I love using Angular's Controller As syntax to clean up my code**, and I'm happy to see this technique [is catching on][3].

Most of the time, I can avoid having to reference `$scope` altogether, but working with forms in Angular has always
felt a little clunky to me. **The FormController kind of just shoves itself into the `$scope`, leaving my controller with
this ugly little reference.**

```javascript
var MyCtrl = function($scope){
  //Yucky reference to $scope in an otherwise
  // beautiful controller
  this.form = $scope.form;
};
```

###Then a miracle happened!

Ok, so not a miracle, but after a bit of googling, I happened across [this pull request from a year ago][1], which ended up
being [merged in here][2].

<h3>Awesome! <small>but totally undocumented...</small></h3>

Now I can do this in my form:

```html
<div ng-controller="MyCtrl as ctrl">

  <!--
    Notice the 'ctrl' prefixing my form name?
    It's the same as my 'Controller As' alias
  -->
  <form name="ctrl.form">
    ...
  </form>

</div>
```

And I now I can reference the form directly on my controller like this:

```javascript
var MyCtrl = function(){
  //Look mom! No $scope!
};

MyCtrl.prototype.save = function(){
  //Glorious $scope free access to FormController!
  if(this.form.$valid){
    ...
  }
};
```

###And there you go.

One more way you can avoid [Scope Soup]({% post_url 2014-03-21-five-guidelines-for-avoiding-scope-soup-in-angular %}) and keep your controllers squeaky clean!

[1]:https://github.com/angular/angular.js/pull/3115
[2]:https://github.com/angular/angular.js/commit/8ea802a1d23ad8ecacab892a3a451a308d9c39d7
[3]:http://toddmotto.com/rethinking-angular-js-controllers/