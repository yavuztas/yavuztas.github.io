---
layout: post
title:  "Containers, Inversion of Control and Dependency Injection"
date:   2019-07-27 14:45:00
readtime: 3 min read
categories: java oop
tags: regular
excerpt: In this article, we will mention <b>containers</b>, <b>inversion of control</b> and <b>dependency injection</b> and give some examples about how they are applied in a simple way.
image: "/assets/article_images/2019-07-27-continers-inversion-of-control-and-dependency-injection/containers-ioc-di.png"
image2: "/assets/article_images/2019-07-27-continers-inversion-of-control-and-dependency-injection/containers-ioc-di-mobile.png"
image-position: right
image2-position: center
---
#### Overview
This is the **second** article of the series [Spring Framework Fundamentals](https://github.com/yavuztas/java-spring-fundamentals){:target="blank"}.

In this article, we will explain **Containers**, what **Inversion of Control** is and how it is implemented by **Dependency Injection** with given examples in a simple way.
### What is Containers?
Containers in computer science, are special structures that contain child objects as well as are responsible for managing their life-cycle. Although there may be different types of containers classified by how they **access**, **store** and **traverse** the objects inside, the most common behavior is, they **create**, **alter** and **remove** these child objects on purpose.

On the wild, this purpose is mostly to be a framework for solving specific business problems or at least providing a way to solve them with ease.

The *ApplicationContext* types from **Spring Framework** is a good example for a container class:
{% highlight java %}

private GenericWebApplicationContext applicationContext;

public void setApplicationContext(GenericWebApplicationContext webApplicationContext){
    this.applicationContext = webApplicationContext;
    // register a bean to context
    this.applicationContext.registerBean(SomeComponent.class, () -> new SomeComponent());
    // and get an instance of that bean
    SomeComponent component = this.applicationContext.getBean(SomeComponent.class);
}

{% endhighlight %}
### What is Inversion of Control?
There is a well-known term as **cohesion** in computer science, which defines the relations of objects and how they can act together in a container class.

Most of the time it is mentioned by **high cohesion**. This means acting well of objects together with great solidarity while keeping complexity low and maintainable.

Besides, while we keep these objects cohesive, we also need to keep them **loosely coupled** because every object should independently operate their own single responsibility.

Hence, this provides some critical benefits such as; **maintainability**, **reusability** and **testability**.

When it comes to **Inversion of Control**, indeed better to call it as a *principle*, a way of implementing highly cohesive and loosely coupled objects. In OOP, it literally means to convert the flow of controlling a child object by delegating its creation logic to another class.

Thus, the parent object does not need to worry about how its dependent objects should be initialized and the coupling becomes loose.

There are many design patterns in order to implement the IoC principle, some of these patterns are *Service Locator*, *Factory*, *Abstract Factory*, *Template Method*, *Strategy*, *Dependency Injection*.

Let's imagine we have a simple *UserService* as a container:
{% highlight java %}
public class UserService {

    private UserDao userDao = new UserDao();

}
{% endhighlight %}
Then, change the code like below by using *Factory* pattern:
{% highlight java %}
public class DaoFactory {

    public static UserDao createUserDao(){
        return new UserDao();
    }

}

public class UserService {

    private UserDao userDao = DaoFactory.createUserDao();

}
{% endhighlight %}
So, we have implemented a basic IoC and made the dependency between *UserService* and *UserDao* less strict which means **loose coupling**.
### What is Dependency Injection?
**Dependency Injection** is a popular design pattern to implement IoC. It is a mechanism that scans object hierarchy, resolves dependencies and assigns the instances properly to dependent objects.

For example in **Spring Framework**, this is clearly provided by the well-known annotation *@Autowired*. Spring's *ApplicationContext* makes a component scan on startup and handles the injection of dependent instances internally among its managed objects.

Let's show a simple DI over a spring bean defined as *UserService*:
{% highlight java %}
@Service
public class UserService implements IUserService {

    @Autowired
    private IUserDao userDao;

}
{% endhighlight %}
Note that we need to depend on abstract types in order to make *loosely coupled* connections. This is also an obvious rule of *Dependency Inversion Principle* which we explained in our [previous article]({{ site.url }}/java/oop/2019/06/22/object-oriented-programming-and-solid-principles.html).

Consequently, we can say that DIP is a well-fit design to implement a practicle *IoC Container*. This is why we use an interface called *IUserDao* instead of a concrete *UserDao* type.

#### Conclusion
After we mentioned about *containers, inversion of control and dependency injection*, in our [next article]({{ site.url }}/java/spring/2019/08/26/application-context-in-spring.html), we will continue with the *ApplicationContext* and explain the basics of *Spring Framework*.
