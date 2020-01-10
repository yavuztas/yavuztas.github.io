---
layout: post
title:  "The BeanDefinitionOverrideException in Spring Boot 2.1"
date:   2020-01-01 14:34:25
categories: java spring-boot
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## 1. Introduction
One of the basic feature the Spring Framework provides is an orchestration of beans. Beyond that, Spring Boot offers us auto-configuration features out of the box to
let us manage our beans in a quick and practical way.

However, it can be tricky to find out the reason of some bean configuration problems without knowing the defaults Spring Boot brings in.

In this tutorial, we'll explain *BeanDefinitionOverrideException* in Spring Boot.

## 2. Bean Overriding in Spring
For each *ApplicationContext* in Spring, beans are identified by their names.

Thus, Bean Overriding is a default behavior that occurs when we have a bean in the same *ApplicationContext* with the same name. It simply replaces the previous bean with the new one in case of a name conflict.

However, Spring 5.1 introduces the new *BeanDefinitionOverrideException*. So that depending on the configuration, Spring can either let bean overriding or throw *BeanDefinitionOverrideException* in order to prevent overriding a bean accidentally. We should be aware of that the default behavior is still the same, allowing bean overriding as we can clearly see *DefaultListableBeanFactory#isAllowBeanDefinitionOverriding()* is set to *true*.

## 3. Configuration Change for Spring Boot 2.1
Another important change we should notice, [Spring Boot 2.1 disabled bean overriding by default](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.1-Release-Notes#bean-overriding) for the sake of defansive approach in bean definitions. So, the main purpose is to notice the conflict in bean definitions in advance and to prevent  overriding beans accidentally.

Therefore, if our Spring Boot application relies on bean overriding, it is very likely to see *BeanDefinitionOverrideException* after we upgrade the Spring Boot version to 2.1 and later.

In next sections, we'll explain how to overcome *BeanDefinitionOverrideException* with the possible workarounds.

## 4. Identifying the Beans in Conflict
- Checking multiple Spring configurations, in properties files or configuration classes or even XML files.

In order to analyze *BeanDefinitionOverrideException*, let's create two different Spring configurations to quickly reproduce the exception:
{% highlight java %}
@Configuration
public class TestConfiguration1 {

    class TestBean1 {
        private String name;
    }

    @Bean
    public TestBean1 testBean(){
        return new TestBean1();
    }

}

@Configuration
public class TestConfiguration2 {

    class TestBean2 {
        private String name;
    }

    @Bean
    public TestBean2 testBean(){
        return new TestBean2();
    }

}
{% endhighlight %}

Next, we create our Spring Boot test class:
{% highlight java %}
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootAppExceptionTests {

    @Autowired
    private ApplicationContext applicationContext;

    @Test
    public void testBeanOverridingIsNotAllowed() {
        Object testBean = applicationContext.getBean("testBean");
        Assert.assertEquals(testBean.getClass(), TestConfiguration2.TestBean2.class);
    }
}
{% endhighlight %}

-EDIT-And, here comes our *BeanDefinitionOverrideException*:
{% highlight java %}
Invalid bean definition with name 'testBean' defined in class path resource [dev/yavuztas/spring/beandefinitionoverrideexception/TestConfiguration2.class]:
Cannot register bean definition [ ... defined in class path resource [dev/yavuztas/spring/beandefinitionoverrideexception/TestConfiguration2.class]] for bean 'testBean':
There is already [ ... defined in class path resource [dev/yavuztas/spring/beandefinitionoverrideexception/TestConfiguration1.class]] bound.
{% endhighlight %}

As we notice, the exception reveals three important information. The first one is the conflicting bean name as *testBean*:
{% highlight java %}
Invalid bean definition with name 'testBean' ...
{% endhighlight %}

Next, we can see the resource type of configuration as *class path resource*:
{% highlight java %}
... defined in class path resource ...
{% endhighlight %}

-EDIT-The last one is the full path of the configurations:
{% highlight java %}
[dev/yavuztas/spring/beandefinitionoverrideexception/TestConfiguration2.class]
[dev/yavuztas/spring/beandefinitionoverrideexception/TestConfiguration1.class]
{% endhighlight %}

## 5. Possible Solutions
Depending on how we define, Spring Beans have default values for their names unless we set names to them explicitly.

Hence, the first possible solution to name our beans uniquely. There are common ways to set bean names in Spring.

### 5.1. Changing Method Names
In Java config

### 5.2. @Bean Annotation
We can set bean name by the *value* property of @Bean annotation

### 5.3. Stereotype Annotations
For component scan=true
- We can explicitly set bean name by the "value" property of stereotype annotations (@Component, @Service, @Repository, @Controller).

### 5.4. Beans Coming From 3rd Party Libraries
In some cases, it's possible to encounter with a name conflict caused by beans originating from 3rd party spring-supported libraries. Certainly, we should identify which conflicting bean belongs to our application to find out that we can apply or not the solutions as we mentioned before.

In any case, if we don't have a chance to alter any of the conflicting bean definitions then configuring *allow-bean-definition-overriding* property can be a workaround.

In order to tell Spring Boot to allow bean overriding, let's add *spring.main.allow-bean-definition-overriding* property to our *application.properties* file:  
{% highlight java %}
spring.main.allow-bean-definition-overriding=true
{% endhighlight %}

## 6. Conclusion
In this tutorial, **we explained what *BeanDefinitionOverrideException* is and showed how to solve it in Spring Boot**.

Like always, the complete source code of this article can be found [over on GitHub](https://github.com).
