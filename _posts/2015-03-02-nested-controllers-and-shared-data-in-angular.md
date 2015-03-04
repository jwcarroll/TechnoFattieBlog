---
layout: post
title: Nested Controllers And Shared Data in Angular
description: Here is a simple technique for sharing data between nested
             controllers without sacrificing design quality.
headerUrl: /img/nested-controllers-code.jpg
tags: [angular, javascript, directives, controllers]
---

##Controllers Should Be Classes

It's a drum I have been beating for close to a year now because I want you to
[avoid creating a mess of $scope soup]({% post_url 2014-03-21-five-guidelines-for-avoiding-scope-soup-in-angular %}).
I'm not alone in this design decision either, as popular style guides by John Papa and Todd Motto advocate for avoiding
the use of $scope inside of controllers in favor of the ever useful [Controller As Syntax][controller-as].

That being said, there are scenarios you will encounter while building your application that seem like using `$scope` is
necessary... maybe even unavoidable in order to solve.

For instance, how do you reference `<form name='myForm'>` without injecting scope? Luckily
[the solution is pretty simple]({% post_url 2014-07-01-using-angular-forms-with-controller-as-syntax %}).

One common scenario I get asked about a lot is how to deal with nested, or parent/child controller scenarios where you
need the child controller to act on data that is *owned* by the parent. This is a good question, and one we can easily solve
without relying on inherited `$scope` values.

##The Problem

As I've already stated, the need for nested controllers is fairly common. Imagine we have a simple contacts controller
that loads a list of contacts from the server, and then displays them in a list.

```javascript
function ContactsController(contactService){
  var _this = this;

  _this.contactService = contactService;
  _this.init();
}

ContactsController.prototype = {
  init:function(){
    var _this = this;

    _this.contactService.getContacts()
      .success(function(contacts){
        _this.contacts = contacts;
      });
  }
};
```

The controller could certainly be much more complex, it could include filtering, or a bulk selection and delete operation,
but at the end of the day, it's primary focus is on the **entire list** of contacts. To keep the concerns separate, we
don't really want the **ContactsController** to be involved managing each individual contact.

This is a perfect place to use a nicely encapsulated controller for each contact, but how do we get the data into the
controller for it to act upon?

##Take That, *Controller As* Syntax!

This is about the time where proponents of using `$scope` inside of the controller feel they have an advantage. It's true
that if we weren't using **Controller As** then it would be trivial to simply reference the local scope property created
by using `ng-repeat`.

```html
<div ng-repeat="contact in contacts">
  <div ng-controller="contactController">
    <!-- some bits of complex DOM here -->
  </div>
</div>
```

But this is awful for all the reasons I have mentioned before:

 - **Implicit Coupling** *(tied to DOM position)*
 - **Magical Initialization** *(where did this value come from?)*
 - **Minimized Reuse** *(try using in a dialog)*

There are two options for us to have our **controller as** cake and eat it too.

##Option 1: Direct Initialization

The easiest possible option for us to get data into a child controller is through simple initialization. The idea being
that we provide a simple initialization function we can call to set our data.

```javascript
function ContactController(){ }

ContactController.prototype = {
  //Initialize the contact on this controller
  setContact: function(contact){
    this.contact = contact;

    //run some code here
  }
};
```

Now we can make use of the `ng-init` directive in order to call this function whenever we bind our controller to the DOM.

```html
<!-- assuming we are using 'controller as' -->
<div ng-repeat="contact in ctrl.contacts"
     ng-controller="contactController as contactCtrl"
     ng-init="contactCtrl.setContact(contact)">
  <!-- some bits of complex DOM here -->
</div>
```

This may seem verbose, but for simple, or one-off scenarios this is actually a pretty good solution. The controller is
still independent of any DOM ordering because we aren't relying on any implicit data being present in scope. Also, the
cognitive model is still pretty easy to reason about because you can see all the markup directly inline.

The disadvantages should be fairly obvious.

We can't easily re-use this without copy-and-paste, and now we have two (or more) places to maintain all that markup.
Make a change in one place, and you have to make sure it's updated in the other(s). Which of course brings us to our second
solution.

##Option 2: Componetize With A Directive

A much better solution would be to simply take our wonderful bit of DOM, our lovely controller, and marry the two together
into a single component. What we really want is something like this in our markup:

```html
<contact ng-repeat="contact in ctrl.contacts" contact="contact"></contact>
```

The hard work has already been done, so all we really have to do is wire up a simple little directive to glue it all together.

```javascript
function ContactDirective(){
  return {
    restrict: 'E',
    templateUrl: 'my-contact-template.html',
    controller: 'contactController',
    controllerAs: 'contactCtrl',
    scope: {
      contact: '='
    },
    link:function(scope, elem, attrs, contactCtrl){
      scope.$watch('contact', function(newContact){
        //Still just initializing the contact using
        // the controller
        contactCtrl.setContact(newContact);
      });
    }
  };
}
```

Some of you may be balking at my use of scope here, but I have said before that I am not against using scope... but **I
*am* against the use of it in controllers in almost all cases.**

The beauty of this approach is that our controller and our template are basically unchanged. The controller still knows
nothing about `$scope`, and **the parent controller is still 100% in charge of the *list of contacts*** like it should be.
The directives sole purpose in life is to package these two together into a neat little component that can be re-used
anywhere in the application.

A full working demo of the latter approach can be seen here.

{% include livedemo.html url="/demos/nested-controllers-shared-data/" %}

##Conclusion

Well, there you have it.

 - Use `ng-init` to set values on your child controller without resorting to scope inheritance.
 - Use directives to wrap controllers and views into re-usable components.

[controller-as]: https://docs.angularjs.org/api/ng/directive/ngController