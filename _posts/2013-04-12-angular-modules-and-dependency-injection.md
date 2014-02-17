---
layout: post
title: Angular Modules And Dependency Injection
tags: [angular, javascript, dependency-injection]
alias: /2013/04/angular-modules-and-dependency-injection.html
---

### Angular is Awesome!

If you aren't already familiar with <a href="http://angularjs.org/">Angular.js</a>, then you really owe it to yourself to dig into this amazing framework. 
<a href="http://misko.hevery.com/">Misko Hevery</a> (<a href="https://twitter.com/mhevery">@mhevery</a>) and his team have done an outstanding job, and I continue to be delighted by just how well designed and flexible the framework is.

### Dependency Injection

At the heart of this flexibility is the <a href="http://docs.angularjs.org/guide/di">Dependency Injection capabilities that Angular provides</a>. However, while this is arguably the most powerful feature in Angular, it can also seem like a bit of voodoo magic at times. You may be wondering just how, or when, you can use this to your advantage.
<b>I'm going to try and shine some light on this subject by showing you the most basic examples of dependency injection in Angular.</b>
Let's imagine we had a simple service like this, and a very simple <a href="http://docs.angularjs.org/guide/module">module</a> that register the service.

{% highlight javascript %}
function simpleService(){
   this.name = "simpleService";
}

angular.module('foo', [])
   .service('simpleService', simpleService);
{% endhighlight %}

### Using The Injector

Now imagine we wanted to manually grab an instance of that service <i>(which you should almost never do)</i> and use it. To do this we can create a new <a href="http://docs.angularjs.org/api/angular.injector">injector </a>and let it do the magic of creating the service for us.

{% highlight javascript %}
var myInjector = angular.injector(['foo']);
var service = myInjector.get('simpleService');

console.log(service.name); // 'simpleService'
{% endhighlight %}

So that's it... the simplest possible case. But what happens if you have two modules with the same service name defined? What happens then? Well, this is where it starts to get interesting. Let's look at two more examples.

{% highlight javascript %}
angular.module('foo', [])
   .service('simpleService', function(){ this.name = "foo"; });

angular.module('bar', [])
   .service('simpleService', function(){ this.name = "bar"; });

var fooSvc = angular.injector(['foo']).get('simpleService');
var barSvc = angular.injector(['bar']).get('simpleService');

console.log(fooSvc.name); // 'foo'
console.log(barSvc.name); // 'bar'
{% endhighlight %}

### Module Ordering

This is more or less what you would expect to see. Each injector references only a single module, and therefore the services are isolated. But can't you create an injector that references more than one module? What happens then?

{% highlight javascript %}
var fooSvc = angular.injector(['foo','bar']).get('simpleService');
var barSvc = angular.injector(['bar','foo']).get('simpleService');

console.log(fooSvc.name); // 'bar'
console.log(barSvc.name); // 'foo'
{% endhighlight %}

Did you see what happened there? <b>Basically the last module to execute and register it's services won. It's simply a matter of the order in which they get executed.</b>
But what if one our modules has a dependency? How does that change things?

{% highlight javascript %}
angular.module('foo', [])
   .service('simpleService', function(){ this.name = "foo"; });

angular.module('bar', ['foo'])
   .service('simpleService', function(){ this.name = "bar"; });

var fooSvc = angular.injector(['foo','bar']).get('simpleService');
var barSvc = angular.injector(['bar','foo']).get('simpleService');

console.log(fooSvc.name); // 'bar'
console.log(barSvc.name); // 'bar'
{% endhighlight %}

### Module Dependencies

Again, it's all about the order in which they execute, but in this particular scenario, because our <b>bar</b> module has a dependency on the <b>foo</b> module, <b>foo</b> will always get invoked first.
This is where you can start to take advantage of this system to change how services operate, or replace them wholesale.
Imagine we were testing, and wanted to replace the <a href="http://docs.angularjs.org/api/ng.$httpBackend">$httpBackend</a> with a custom one, or a mock implementation that we can precisely control?
This becomes trivially easy to do because of how Angular is designed.
All you would have to do is create a module that is dependent on the <a href="http://docs.angularjs.org/api/ng">ng module</a>, and override any service you wanted to.
Indeed this is exactly how the <a href="http://docs.angularjs.org/api/ngMock">ngMock module</a> works.

### Service Interception

Now, replacing a service is great, but one especially nice feature in Angular is the ability to intercept a service just after it has been created. This can be achieved easily by using a decorator.

{% highlight javascript %}
angular.module('foo', [])
   .service('simpleService', function(){ this.name = "foo"; });

angular.module('bar', ['foo'])
   .config(function($provide){

      //$provide was injected for me automatically by Angular
      $provide.decorator('simpleService', function($delegate){

         //$delegate is the service instance, and is
         // also injected automatically by Angular
         $delegate.name += "|bar";

  return $delegate;
      });
   });

var fooSvc = angular.injector(['foo']).get('simpleService');
var barSvc = angular.injector(['bar']).get('simpleService');

console.log(fooSvc.name); // 'foo'
console.log(barSvc.name); // 'foo|bar'
{% endhighlight %}

### Conclusion

I won't go into all the gory details on this, but basically we are able to intercept the <b>simpleService</b> from <b>foo</b> and do something with it before returning. You can check out the documentation on both the <a href="http://docs.angularjs.org/api/angular.Module">config</a> and the <a href="http://docs.angularjs.org/api/AUTO.$provide#decorator">decorator</a> methods.
I hope this sheds a little light on modules, services and dependency injection in Angular. The ability to replace and intercept services opens up a whole world of possibilities and makes testing much easier.
