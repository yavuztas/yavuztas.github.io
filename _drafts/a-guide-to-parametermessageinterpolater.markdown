---
layout: post
title:  "A Guide to ParameterMessageInterpolator"
date:   2019-10-15 14:00:00
categories: java bean-validation
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## Questions to editor?
* We can consider putting a link after the error message also from javax-validation article to this article...

## 1. Overview
In this short article, we'll have a look at how to configure *ParameterMessageInterpolator* in Hibernate Validator.

One of the features of Java JSR 380, also known as *Bean Validation 2.0*, is allowing expressions while interpolating the validation messages with parameters.

When we use Hibernate Validator as the reference implementation, to support parsing of these expressions, there is a prerequisite that we need to add one of the uniform implementations of Java Expression Language Api, also called JSR 341, as an extra dependency to our project.

However, adding an additional dependency can be cumbersome if we don't really need it according to the use case.

## 2. Message Interpolators
Beyond [the basics of validating a Java bean](https://www.baeldung.com/javax-validation), Bean Validation Api's *MessageInterpolator* is an abstraction which brings us a way of performing simple message interpolations without the hassle of parsing expressions.

For the sake of simplicity, we can relief our project dependencies from expression language api with Hibernate Validator's non-expression based *ParameterMessageInterpolator* which offers us a super basic parameter interpolation.

## 2. Setting up Custom Message Interpolators
In order to configure Hibernate Validator without expression language support, we can use custom message interpolators.

In the next two sections, we are going to show convenient ways of plugging in any custom *MessageInterpolator* implementation, which will be the built-in *ParameterMessageInterpolator* in our example.

### 2.1. Configuring *ValidatorFactory*
One way of using a custom message interpolator is configuring *ValidatorFactory*.

Let's initialize a *ValidatorFactory* instance with *ParameterMessageInterpolator:*
{% highlight java %}
ValidatorFactory validatorFactory = Validation.byDefaultProvider()
  .configure()
	.messageInterpolator(new ParameterMessageInterpolator())
	.buildValidatorFactory();
{% endhighlight %}

### 2.2. Configuring *Validator*
Similarly, we can configure *Validator* instance instead of *ValidatorFactory:*
{% highlight java %}
ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();

Validator validator = validatorFactory.usingContext()
	.messageInterpolator(new ParameterMessageInterpolator())
	.getValidator();
{% endhighlight %}

## 3. Performing Validations
In order to see how *ParameterMessageInterpolator* performs message interpolation, we need a sample java bean configured with some JSR 380 annotations.

### 3.1. Sample Java Bean
Let's define our simple java bean *Person:*
{% highlight java %}
public class Person {

	@NotNull(message = "Name cannot be null")
	private String name;

	@Size(min = 10, max = 200, message = "Biography should be in-between {min} and {max} characters")
	private String biography;

	@Min(value = 18, message = "Age should not be less than {value} but it is ${validatedValue}")
	private int age;

	// standard getters and setters

}
{% endhighlight %}

### 3.2. Integration Tests
We can perform our validations by simply using *Validator* instance accessed from *ValidatorFactory* which we already created with *ParameterMessageInterpolator:*
{% highlight java %}
Validator validator = validatorFactory.getValidator();
{% endhighlight %}

First, we run our test for *biography* field:
{% highlight java %}
@Test
public void ifBioLengthIsLess_bioValidationFails() {
	Person person = new Person();
	person.setName("John Doe");
	person.setBiography("short bio");
	person.setAge(18);

	Set<ConstraintViolation<Person>> violations = validator.validate(person);
	assertEquals(1, violations.size());

	ConstraintViolation<Person> violation = violations.iterator().next();
	assertEquals("Biography should be in-between 10 and 200 characters", violation.getMessage());
}
{% endhighlight %}
The output of validation message will be interpolated with variables of *min* and *max* correctly:
{% highlight java %}
Biography should be in-between 10 and 200 characters
{% endhighlight %}
Then, we run a similar test for another field *age*:
{% highlight java %}
@Test
public void ifAgeIsLess_ageMinValidationFails() {
	Person person = new Person();
	person.setName("John Doe");
	person.setAge(16);

	Set<ConstraintViolation<Person>> violations = validator.validate(person);
	assertEquals(1, violations.size());

	ConstraintViolation<Person> violation = violations.iterator().next();
	assertEquals("Age should not be less than 18 but it is 16", violation.getMessage());
}
{% endhighlight %}
This time the output will be:
{% highlight java %}
Age should not be less than 18 but it is ${validatedValue}
{% endhighlight %}
As we notice, the variable *value* is interpolated but the expression *validatedValue* is not.

We should take account of that *ParameterMessageInterpolator* only supports the interpolation of message parameters, not parsing expressions which are stated with an additional *${...}* notation. Instead, it simply returns them un-interpolated.

## 5. Conclusion
In this tutorial, **we learned what *ParameterMessageInterpolator* is for and how to configure it in Hibernate Validator.**

As always, all the examples involved in this tutorial are available [over on GitHub](https://github.com).
