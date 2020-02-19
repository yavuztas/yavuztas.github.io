---
layout: post
title:  "The BeanDefinitionOverrideException in Spring Boot"
date:   2020-01-01 14:34:25
categories: java spring-boot
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## 1. Introduction
Although defining beans can be pretty straightforward, it can be difficult to determine the cause of some configuration problems without knowing the defaults Spring Boot provides. A common problem is the *BeanDefinitionOverrideException* that can confuse some developers about the *Bean Overriding* behavior in Spring.

In this tutorial, we’ll take a look at Spring’s BeanDefinitionOverrideException and how to handle it.

--- version 2 ---

The Spring Boot 2.1 upgrade surprised a number of people with unexpected occurances of *BeanDefinitionOverrideException*. It can also confuse some developers and make them wondering about what happened to the *Bean Overriding* behavior in Spring.

In this tutorial, we'll unravel this issue and then see how best to address it.

## 2. Bean Overriding in Spring
For each *ApplicationContext* in Spring, beans are identified by their names.

Thus, **Bean Overriding is a default behavior that happens when we have a bean in the same *ApplicationContext* with the same name**. It works by simply replacing the former bean in case of a name conflict.

However, Spring 5.1 introduces the new [*BeanDefinitionOverrideException*](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionOverrideException.html). So that **depending on the configuration, Spring can either let bean overriding or throw *BeanDefinitionOverrideException* to prevent overriding a bean accidentally**.

Besides, we should be aware that the default behavior is still the same, allows bean overriding as we can see [*DefaultListableBeanFactory#setAllowBeanDefinitionOverriding()*](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/beans/factory/support/DefaultListableBeanFactory.html#setAllowBeanDefinitionOverriding-boolean-) is set to *true*.

## 3. Configuration Change for Spring Boot 2.1
Another important change we should notice, [Spring Boot 2.1 disabled bean overriding by default](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.1-Release-Notes#bean-overriding) for the sake of defensive approach in bean definitions. So, **the main purpose is to notice the conflict in bean definitions in advance and to prevent overriding beans accidentally**.

Therefore, if our Spring Boot application relies on bean overriding, it is very likely to see *BeanDefinitionOverrideException* after we upgrade the Spring Boot version to 2.1 and later.

In the next sections, we'll explain how to overcome *BeanDefinitionOverrideException* with the possible workarounds.

## 4. Identifying the Beans in Conflict
In order to analyze, let's create two different Spring configurations to quickly reproduce *BeanDefinitionOverrideException*:
{% highlight java %}
@Configuration
public class TestConfiguration1 {

    class TestBean1 {

        private String name;

        // standard getters and setters

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

        // standard getters and setters

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
public class SpringBootBeanDefinitionOverrideExceptionIntegrationTest {

    @Autowired
    private ApplicationContext applicationContext;

    @Test
    public void whenBeanOverridingAllowed_thenTestBean2OverridesTestBean1() {
        Object testBean = applicationContext.getBean("testBean");

        assertThat(testBean.getClass()).isEqualTo(TestConfiguration2.TestBean2.class);
    }
}
{% endhighlight %}

So, here comes our *BeanDefinitionOverrideException*:
{% highlight java %}
Invalid bean definition with name 'testBean' defined in com.baeldung.beandefinitionoverrideexception.TestConfiguration2 ...
Cannot register bean definition [ ... defined in com.baeldung.beandefinitionoverrideexception.TestConfiguration2] for bean 'testBean' ...
There is already [ ... defined in com.baeldung.beandefinitionoverrideexception.TestConfiguration1] bound.
{% endhighlight %}

As we notice, the exception reveals two important pieces of information.

The former one is the conflicting bean name as *testBean*:
{% highlight java %}
Invalid bean definition with name 'testBean' ...
{% endhighlight %}

And the latter, we can get the full path of the configurations:
{% highlight java %}
... com.baeldung.beandefinitionoverrideexception.TestConfiguration2 ...
... com.baeldung.beandefinitionoverrideexception.TestConfiguration1 ...
{% endhighlight %}

As a result, we can figure out that we have two different beans point out the same *testBean* name inside the configuration classes *TestConfiguration1* and *TestConfiguration2*.

## 5. Possible Solutions
Depending on how we define, Spring Beans have default values for their names unless we set names to them explicitly.

Certainly, the first possible solution to rename our beans uniquely. There are common ways to set bean names in Spring.

### 5.1. Changing Method Names
Spring sets the name of the methods as bean names by default for Java configuration.

Therefore, if we have beans defined in a Java configuration file, like our example, then simply changing the method names can be an option:
{% highlight java %}
@Bean
public TestBean1 testBean1(){
    return new TestBean1();
}

@Bean
public TestBean1 testBean2(){
    return new TestBean2();
}
{% endhighlight %}

### 5.2. @Bean Annotation
Of course, using Spring's *@Bean* annotation is super common when it comes to setting bean names.

Thus, another option is to set the *name* property of *@Bean* annotation:
{% highlight java %}
@Bean("testBean1")
public TestBean1 testBean(){
    return new TestBean1();
}

@Bean("testBean2")
public TestBean1 testBean(){
    return new TestBean2();
}
{% endhighlight %}


### 5.3. Stereotype Annotations
Moreover, another common way to define Spring beans is [stereotype annotations](https://www.baeldung.com/spring-bean-annotations). So, we can also specify bean names by using them.

Since Spring's *@ComponentScan* feature is enabled by default in Spring Boot, we can define our beans by *@Component* annotation:
{% highlight java %}
@Component("testBean1")
class TestBean1 {

    private String name;

    // standard getters and setters

}

@Component("testBean2")
class TestBean2 {

    private String name;

    // standard getters and setters

}
{% endhighlight %}

### 5.4. Beans Coming From 3rd Party Libraries
In some cases, **it's possible to encounter a name conflict caused by beans originating from 3rd party spring-supported libraries**.

Obviously, we should identify first which conflicting bean belongs to our application, in order to find out that we can apply or not the solutions as we mentioned before.

In any case, if we don't have a chance to alter any of the bean definitions, then setting *allow-bean-definition-overriding* property can be a workaround.

In order to configure Spring Boot to allow bean overriding, let's add *spring.main.allow-bean-definition-overriding* property to our *application.properties* file:  
{% highlight java %}
spring.main.allow-bean-definition-overriding=true
{% endhighlight %}

As a result, Spring allows bean overriding without any need to change our beans.

## 6. Conclusion
In this tutorial, **we explained what *BeanDefinitionOverrideException* is, when it happens and how to solve it in Spring Boot**.

Like always, the complete source code of this article can be found [over on GitHub](https://github.com).
