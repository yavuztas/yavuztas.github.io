---
layout: post
title:  "The BeanDefinitionOverrideException in Spring Boot 2.1"
date:   2020-01-01 14:34:25
categories: java spring-boot
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## Notes
- Check we can set a name to also application context, maybe the null refers to context name like null for root or when we don't set it:
"The bean 'agentRepository', defined in null, could not be registered. A bean with that name has already been defined in null and overriding is disabled"
This might come up with another solution!

## 1. Introduction
- Define what is BeanDefinitionOverrideException in Spring

## 2. Bean Overriding in Spring
- Explain what bean overriding is in Spring
- Tell the scenarios when it may occur
- Two beans with the same name
- Beans coming from 3rd party libraries  

## 3. How to Reproduce?
- A sample code to reproduce the exception

## 4. Configuration Change for Spring Boot 2.1
- Mention about the change (normally false in Spring Beans Module but configured true in Spring Boot)
- Give the link of relevant release notes
- Tell why Spring changed this default behavior

## 5. Renaming the Beans in Conflict
- Show possible ways of bean renaming in Spring Java config

### 5.1. Changing Method Names

### 5.2. @Bean Annotation
We can set bean name by the *value* property of @Bean annotation

### 5.3. Stereotype Annotations
- We can explicitly set bean name by the "value" property of stereotype annotations (@Component, @Service, @Repository, @Controller).

## 6. Enable Bean Overriding
- Show override bean configuration

## 7. Conclusion


## 2. Sample Java Code
This is a sample code block:
{% highlight java %}
public class Java {

}
{% endhighlight %}

## 7. Conclusion
In this tutorial, **we learned something super cool :)**.

As always, the full source code demonstrated in the examples for this tutorial are available [over on GitHub](https://github.com).
