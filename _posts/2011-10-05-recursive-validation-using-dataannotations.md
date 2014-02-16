---
layout: post
title: Recursive Validation Using DataAnnotations
tags: [c#,dot-net,validation]
---

I saw a <a href="http://stackoverflow.com/questions/7663501/dataannotations-recursively-validating-an-entire-object-graph">post on Stackoverflow</a> today that piqued my interest.

The title really says it all:

<span class="Apple-style-span" style="background-color: white; font-family: 'Trebuchet MS', 'Liberation Sans', 'DejaVu Sans', sans-serif; line-height: 12px;"></span>
<h1 style="background-attachment: initial; background-clip: initial; background-color: transparent; background-image: initial; background-origin: initial; border-bottom-width: 0px; border-color: initial; border-left-width: 0px; border-right-width: 0px; border-style: initial; border-top-width: 0px; font-family: 'Trebuchet MS', 'Liberation Sans', 'DejaVu Sans', sans-serif; font-weight: bold; margin-bottom: 7px; margin-left: 0px; margin-right: 0px; margin-top: 0px; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; vertical-align: baseline;"> <span class="Apple-style-span" style="background-attachment: initial; background-clip: initial; background-color: transparent; background-image: initial; background-origin: initial; border-bottom-width: 0px; border-color: initial; border-left-width: 0px; border-right-width: 0px; border-style: initial; border-top-width: 0px; color: black; cursor: pointer; font-size: small; font-weight: bold; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-decoration: none; vertical-align: baseline;"><a class="question-hyperlink" href="http://stackoverflow.com/questions/7663501/dataannotations-recursively-validating-an-entire-object-graph" style="background-attachment: initial; background-clip: initial; background-color: transparent; background-image: initial; background-origin: initial; border-bottom-width: 0px; border-color: initial; border-left-width: 0px; border-right-width: 0px; border-style: initial; border-top-width: 0px; color: black; cursor: pointer; font-weight: bold; margin-bottom: 0px; margin-left: 0px; margin-right: 0px; margin-top: 0px; padding-bottom: 0px; padding-left: 0px; padding-right: 0px; padding-top: 0px; text-decoration: none; vertical-align: baseline;">DataAnnotations: Recursively validating an entire object graph</a></span></h1>
Using the DataAnnotations to do validation on object can be extermely useful, but there are many limitations to the existing code base. One such limitations, as <a href="http://stackoverflow.com/users/26414/neil-barnwell">Neil </a>has discovered, is that the out of the box validator doesn't recursively apply validation.

Indeed looking at the MSDN docs for <a href="http://msdn.microsoft.com/en-us/library/dd411772.aspx">TryValidateObject</a> state clearly that:
<blockquote><span class="Apple-style-span" style="font-family: 'Segoe UI', Verdana, Arial; font-size: 13px;">This method evaluates each&nbsp;<a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.validationattribute.aspx" style="color: #1364c4; text-decoration: none;">ValidationAttribute</a>&nbsp;instance that is attached to the object type. It also checks whether each property that is marked with&nbsp;<a href="http://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.requiredattribute.aspx" style="color: #1364c4; text-decoration: none;">RequiredAttribute</a>&nbsp;is provided.<span class="Apple-style-span" style="background-color: yellow;"> It does not recursively validate</span> the property values of the object.</span></blockquote>&nbsp;So how do we get around this? Well... there are several examples of people using reflection and LINQ to recurse over the individual properties and validate each one of them. Yuck!

<b>I'm a big fan of working with the framework</b> if at all possible, so I decided to implement a quick solution that fits nicely into the DataAnnotations world.

The idea is to simply tell the validation framework that you would like to do a little extra with some particular property. I want to be able to be able to <i>opt in</i> certain properties to further validation if necessary. Something like this.

{% highlight c# %}
public class Person {
  [Required]
  public String Name { get; set; }

  [Required, ValidateObject]
  public Address Address { get; set; }
}
{% endhighlight %}
Address could have it's own set of validation attributes, and I want to also validate those along with my Person type. <b>Turns out this is pretty simple to do using existing framework components.</b>

Let me just show you the code, and then I can explain what is going on.

{% highlight c# %}
public class ValidateObjectAttribute: ValidationAttribute {
   protected override ValidationResult IsValid(object value, ValidationContext validationContext) {
      var results = new List<ValidationResult>();
      var context = new ValidationContext(value, null, null);

      Validator.TryValidateObject(value, context, results, true);

      if (results.Count != 0) {
         var compositeResults = new CompositeValidationResult(String.Format("Validation for {0} failed!", validationContext.DisplayName));
         results.ForEach(compositeResults.AddResult);

         return compositeResults;
      }

      return ValidationResult.Success;
   }
}

public class CompositeValidationResult: ValidationResult {
   private readonly List<ValidationResult> _results = new List<ValidationResult>();

   public IEnumerable<ValidationResult> Results {
      get {
         return _results;
      }
   }

   public CompositeValidationResult(string errorMessage) : base(errorMessage) {}
   public CompositeValidationResult(string errorMessage, IEnumerable<string> memberNames) : base(errorMessage, memberNames) {}
   protected CompositeValidationResult(ValidationResult validationResult) : base(validationResult) {}

   public void AddResult(ValidationResult validationResult) {
      _results.Add(validationResult);
   }
}
{% endhighlight %}

The idea here is pretty simple. Any property decorated with the ValidateObjectAttribute will be validated as a whole, and the results aggregated together. The processing is as follows: 

1. Create a new ValidationContext using the current value as the target
2. Attempt to validate that object, being sure to include all properties
3. If there are any ValidationResults, store them and return as a single aggregate failure
4. If there are no results, return ValidationResults.Success

Putting it all together, imagine for a minute that we had the following class structure:

{% highlight c# %}
public class Person {
  [Required]
  public String Name { get; set; }

  [Required, ValidateObject]
  public Address Address { get; set; }
}

public class Address {
  [Required]
  public String Street1 { get; set; }

  public String Street2 { get; set; }

  [Required]
  public String City { get; set; }

  [Required]
  public String State { get; set; }

  [Required, ValidateObject]
  public ZipCode Zip { get; set; }
}

public class ZipCode {
  [Required]
  public String PrimaryCode { get; set; }

  public String SubCode { get; set; }
}
{% endhighlight %}

We can simply make use of existing methods and validate the entire object graph. Also, making use of our CompositeValidationResult, we should be able to drill into each individual failure in context.

{% highlight c# %}
static void Main(string[] args) {
   var person = new Person {
      Address = new Address {
         City = "Awesome Town",
         State = "TN",
         Zip = new ZipCode()
      },
      Name = "Josh"
   };

   var context = new ValidationContext(person, null, null);
   var results = new List<ValidationResult>();

   Validator.TryValidateObject(person, context, results, true);

   PrintResults(results, 0);

   Console.ReadKey();
}

private static void PrintResults(IEnumerable<ValidationResult> results, Int32 indentationLevel) {
   foreach (var validationResult in results) {
      SetIndentation(indentationLevel);

      Console.WriteLine(validationResult.ErrorMessage);
      Console.WriteLine();

      if (validationResult is CompositeValidationResult) {
         PrintResults(((CompositeValidationResult)validationResult).Results, indentationLevel + 1);
      }
   }
}

private static void SetIndentation(int indentationLevel) {
   Console.CursorLeft = indentationLevel * 4;
}
{% endhighlight %}

Running the above should result in the following output: 

<div class="separator" style="clear: both; text-align: center;"><a href="/img/RecursiveValidation.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" height="163" src="http://1.bp.blogspot.com/-mJLDCk0nY4Y/Toycmc6xuWI/AAAAAAAABF8/M99mCHMfxWs/s400/RecursiveValidation.png" width="366" /></a></div>

Well, there you have it. A pretty simple solution that has the advantage of working alongside the Validation framework instead of against it!
