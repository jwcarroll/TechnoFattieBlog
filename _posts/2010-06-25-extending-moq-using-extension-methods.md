---
layout: post
title: Extending Moq Using Extension Methods
tags: [c#,.net,tdd,unit-testing]
---

If you regularly use <a href="http://en.wikipedia.org/wiki/Test-driven_development">TDD</a>, then you are familiar with the concept of mocking. If you don't use TDD, then you need to slap yourself and apologize to <a href="http://www.objectmentor.com/omTeam/martin_r.html">Bob Martin</a>, <a href="http://www.objectmentor.com/omTeam/feathers_m.html">Michael Feathers</a> and <a href="http://www.objectmentor.com/omTeam/jeffries_r.html">Ron Jeffries</a> immediately. I'll wait here...

While I would argue you should begin your TDD journey by <a href="http://blog.objectmentor.com/articles/2009/10/28/manual-mocking-resisting-the-invasion-of-dots-and-parentheses">writing your own mocks</a>; using a framework will certainly make like easier in many instances and allow you to focus on the tests themselves.

I was a <a href="http://www.ayende.com/projects/rhino-mocks.aspx">RhinoMocks</a> guy for a long time, but have recently converted to <a href="http://code.google.com/p/moq/">Moq</a>... AND I LOVE IT!!! Moq is simple and easy to use, and doesn't get in the way of the intent of your tests. That being said, there are certain scenarios you run into with any framework that you end up coding around, and sometimes that <em>does</em> get in the way of your what you trying to say with your test.

Moq allows for a pretty simple syntax when you want to set up a return value on a mocked type.

{% highlight c# %}
var mockService = new Mock<IReturnStuff>();
mockService.Setup(svc => svc.SomeMethodCall()).Returns("Hello, World!");
{% endhighlight %}

Pretty sweet so far, and that works 99% of the time. But... what about a scenario where we want to return different values on successive calls to that method. I can think of a lot of situations where, in the absence of formal eventing, you want your code to test some value until it meets a certain criteria. You might be polling a web service for status updates, or checking a database to see if a record has been written. Whatever the case may be, it <em>will</em> come up.

So, for illustration purposes, let's make up one such example. Suppose we wanted to ask a service for a specified set of values, but the return types could be varied. Perhaps we only care about some subset of those values, and want to test that our code correctly deals with variable data.

We have our service contract:

{% highlight c# %}
public interface IReturnStuff{
   String ReturnAString();
}
{% endhighlight %}

Next we have our consuming class (the subject under test):

{% highlight c# %}
public class UsesReturnedStuff{
    public IReturnStuff ReturnStuff { get; set; }
    private Regex Filter { get; set; }

    public UsesReturnedStuff(IReturnStuff returnStuff){
        ReturnStuff = returnStuff;
        Filter = new Regex("(the|quick|brown|fox|hello|world)(:Pu)?", RegexOptions.IgnoreCase);
    }

    public List<String> GetStuff(Int32 times)
    {
        var stuff = new List<String>();

        for (var i = 0; i > times; i++)
        {
            var newStuff = ReturnStuff.ReturnAString();

            if(Filter.IsMatch(newStuff))
                stuff.Add(newStuff);
        }

        return stuff;
    }
}
{% endhighlight %}

As you can see we have a pretty basic setup here that uses a white-list approach to building up a list. We want to write some tests to mock this behavior. If we wanted two different return values when this method is called, we might instinctively write a test like this:


{% highlight c# %}
[Test]
public void LastSetupWins()
{
    var mockReturnStuff = new Mock<IReturnStuff>();
    var usesStuff = new UsesReturnedStuff(mockReturnStuff.Object);

    mockReturnStuff.Setup(svc => svc.ReturnAString()).Returns("Hello");
    mockReturnStuff.Setup(svc => svc.ReturnAString()).Returns("World!");

    var stuff = usesStuff.GetStuff(2);

    Assert.AreEqual("Hello World!", String.Join(" ", stuff.ToArray()));
}
{% endhighlight %}

If you couldn't already tell from the name of the test method, this won't work as the last return value to be Setup wins. The output from this test will produce the text:
<blockquote>"World! World!"</blockquote>Not so great, but the Moq documentation gives us a workaround by using a callback method. A little bit of refactoring and Voila!

{% highlight c# %}
[Test]
public void SetupWithCallback()
{
    var mockReturnStuff = new Mock<IReturnStuff>();
    var usesStuff = new UsesReturnedStuff(mockReturnStuff.Object);

    var returnValues = new[] {"Hello", "World!"};
    var numCalls = 0;

    mockReturnStuff.Setup(svc => svc.ReturnAString())
        .Returns(() => returnValues[numCalls])
        .Callback(() => numCalls++);

    var stuff = usesStuff.GetStuff(returnValues.Length);

    Assert.AreEqual("Hello World!", String.Join(" ", stuff.ToArray()));
}
{% endhighlight %}

Here we are taking advantage of anonymous methods and closures to essentially keep feeding our Mock service it's next value. This works, but... yuck! Reading this test is not very clear, and it gets in the way of the intent.
Since we are supposed to refactor our test code too, lets move the ugly bits out into a private method, and re-write our test again.


{% highlight c# %}
private static Int32 SetupMany(Mock<IReturnStuff> mock, Expression<Func<IReturnStuff, String>> expression, params String[] args)
{
    var numCalls = 0;

    mock.Setup(expression)
        .Returns(() => args[numCalls])
        .Callback(() => numCalls++);

    return args.Length;
}

[Test]
public void SetupWithExtractedMethod()
{
    var mockReturnStuff = new Mock<IReturnStuff>();
    var usesStuff = new UsesReturnedStuff(mockReturnStuff.Object);

    var calls = SetupMany(mockReturnStuff,
                          svc => svc.ReturnAString(),
                          "Hello", "World!");

    var stuff = usesStuff.GetStuff(calls);

    Assert.AreEqual("Hello World!", String.Join(" ", stuff.ToArray()));
}
{% endhighlight %}

This is good. We haven't lost any functionality, but the intent of the test is much clearer. There is less friction because you can read the code and understand what it is doing pretty quickly without having to "decipher" the meaning.
Now, this is all fine and good, but I am likely to run into this same scenario more than once. What I would really love to have is a SetupMany() method that was built into Moq. A method I could call directly on the mocked object. Well... starting with C# 3.0 Extension Methods give us the ability to do just that. With a little work, we should be able to create a very generic extension to give us the desired behavior.


{% highlight c# %}
public static class MoqExtensions
{
    public static void SetupMany<TSvc, TReturn>(this Mock<TSvc> mock,
        Expression<Func<TSvc, TReturn>> expression,
        params TReturn[] args)
        where TSvc : class
    {
        var numCalls = 0;

        mock.Setup(expression)
            .Returns(() => numCalls < args.Length ? args[numCalls] : args[args.Length - 1])
            .Callback(() => numCalls++);
    }
}
{% endhighlight %}

Ok, so I know the method signature looks pretty scary. Just don't stare at it long enough or you will go blind! The extra bit with the Ternary operator is a bit of error checking to prevent you from running over the bounds of the array. If you call the method more times than available arguments, it will simply keep returning the last argument in the list.
Refactoring our test one last time gives us an even cleaner syntax that works with any return type. The second test merely demonstrates the "sticky" final value being returned over and over.


{% highlight c# %}
[Test]
public void SetupWithGenericExtensionMethod()
{
    var mockReturnStuff = new Mock&lt;IReturnStuff&gt;();
    var usesStuff = new UsesReturnedStuff(mockReturnStuff.Object);

    mockReturnStuff.SetupMany(svc =&gt; svc.ReturnAString(), "Hello", "World!");

    var stuff = usesStuff.GetStuff(2);

    Assert.AreEqual("Hello World!", String.Join(" ", stuff.ToArray()));
}

[Test]
public void SetupWithLotsOfParams()
{
    var mockReturnStuff = new Mock<IReturnStuff>();
    var usesStuff = new UsesReturnedStuff(mockReturnStuff.Object);

    mockReturnStuff.SetupMany(svc => svc.ReturnAString(),
                              "the","quick","brown","fox","jumps","over","the","fence","hello!");

    var stuff = usesStuff.GetStuff(11);

    Assert.AreEqual("the quick brown fox the hello! hello! hello!", String.Join(" ", stuff.ToArray()));
}
{% endhighlight %}

And there you have it. Extending the Moq API without having to actually go in and modify the source. Hopefully this will save you a headache or two in the future.