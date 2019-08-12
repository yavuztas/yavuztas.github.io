---
layout: post
title:  "ApplicationContext in Spring Framework"
date:   2019-07-26 19:10:00
categories: java spring
excerpt: In this post, we are going to explain <i>ApplicationContext</i> in <b>Spring Framework</b>.
image: "/assets/images/header-spring.png"
image2: "/assets/images/header-spring-mobile.png"
---
### Overview
This is the **third** post of the series [Spring Framework Fundamentals](https://github.com/yavuztas/java-spring-fundamentals){:target="blank"}.

In this post, we are going to explain *ApplicationContext* in **Spring Framework**.
### What is meant by *ApplicationContext*?
*ApplicationContext* is a central Spring container that provides many ready to use business features for enterprise-level applications. The basic service of *ApplicationContext* is exposing another container **BeanFactory**, which are managing the lifecycle of a **Spring Bean** such as creating, wiring, assembling and accessing them up to their destruction. *ApplicationContext* is a higher-level container than *BeanFactory*, already contains its features by implementing the various *BeanFactory* interfaces.

Besides these bean-related abilities, *ApplicationContext* provides more features like;
 * Providing application level configurations
 * Reading textual messages from convenient *MessageSource* properties
 * Publishing application events
 * Providing automatic post processor registrations like *BeanPostProcessor* and *BeanFactoryPostProcessor*.

On some occasions, like mobile environments where you have limited memory resources and performance requirements, you may consider using *BeanFactory* instead of *ApplicationContext*. However, the latter is more appropriate and we prefer for server-based applications in most cases.

You can check the description [over spring java doc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html) for more details.
### Creating an *ApplicationContext*
Spring Framework has many special *ApplicationContext* implementations for different environments but creating a new *ApplicationContext* is as simple as initializing just a simple java class.

Let's give an example of creating a *FileSystemXmlApplicationContext* instance:
{% highlight java %}
public class SampleApp {

	public static void main(String[] args) {

		FileSystemXmlApplicationContext appContext = new FileSystemXmlApplicationContext("src/main/resources/beans.xml");

		SampleBean sampleBean = appContext.getBean("sampleBean", SampleBean.class);
		sampleBean.saySomething("Hello World!");

	}

}
{% endhighlight %}
Also, there are different *ApplicationContext* instances like:
{% highlight java %}

ClassPathXmlApplicationContext appContext = new ClassPathXmlApplicationContext("beans.xml");

SampleBean sampleBean = appContext.getBean("sampleBean", SampleBean.class);
sampleBean.saySomething("Hello World!");

{% endhighlight %}
And this is the XML file for our configuration:
{% highlight xml %}
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

	<bean id="sampleBean" class="dev.yavuztas.spring.context.SampleBean">
		<property name="message" value="Initial value" />
	</bean>

</beans>
{% endhighlight %}
### Lifecycle of a Spring Bean in an *ApplicationContext*
Spring beans are managed and orchestrated by the IoC container *BeanFactory* which is already available in *ApplicationContext*. There are many stages for a spring bean from creation to destruction when we go through deep however, we keep it simple by grouping into 5 sections.

These sections are;
 * *Aware* interfaces for injecting infrastructure components
 * *@PostConstruct* and *@PreDestroy* annotations
 * *InitializingBean* and *DisposableBean* for callback execution
 * *init-method* and *destroy-method* configurations
 * *BeanPostProcessor* for altering beans before or after initialization

< PUT AN IMAGE SHOWING AS DIAGRAM >

As shown in the diagram after constructor execution and setting properties, **first of all** *Aware* interfaces come like *BeanNameAware*, *BeanFactoryAware*.

Let's give an example of how to use these *Aware* interfaces:
{% highlight java %}
public class SampleBean implements BeanNameAware, BeanFactoryAware {

	private String message;

	@Override
	public void setBeanName(String name) {

		System.out.println("--- setBeanName executed ---");

	}

	@Override
	public void setBeanFactory(BeanFactory beanFactory) throws BeansException {

		System.out.println("--- setBeanFactory executed ---");

	}

    // getters and setters

}
{% endhighlight %}
When we initialize the spring context we will see that *setBeanName* method executed before the *setBeanFactory* method. This indicates that there is also an execution order between *Aware* interfaces. For more details and a full list of them, you may check over on [spring docs](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/Aware.html)

For the **second** step, the bean methods which are marked by *@PostConstruct* annotation are executed.

Let's define another method for our bean configured as *@PostConstruct*:
{% highlight java %}

@PostConstruct
public void postConstruct() {

	System.out.println("--- @PostConstruct executed ---");

}

{% endhighlight %}
To make *@PostConstruct* and *@PreDestroy* annotations work, we need to define *CommonAnnotationBeanPostProcessor* as registered in our spring configuration. Another way to make it out is configuring *context:annotation-config* either in XML or Java configuration.
{% highlight xml %}
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

	<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor" />

	<bean id="sampleBean" class="dev.yavuztas.spring.context.SampleBean">
		<property name="message" value="Initial value" />
	</bean>

</beans>
{% endhighlight %}
As the **third** step, *InitializingBean* interface comes in with its single *afterPropertiesSet* callback function.

Let's make our bean implements *InitializingBean*:
{% highlight java %}
public class SampleBean implements BeanNameAware, BeanFactoryAware, InitializingBean {

    // rest of the class omitted

	@Override
	public void afterPropertiesSet() throws Exception {

		System.out.println("--- afterPropertiesSet executed ---");

	}

}
{% endhighlight %}

As the step **four**, *init-method* configuration shows up. We can define any method and mark it in our bean configuration.

Let's create another method for our bean:
{% highlight java %}

public void initMethod() {

	System.out.println("--- init-method executed ---");

}

{% endhighlight %}
And mark it as *init-method*:
{% highlight xml %}

<bean id="sampleBean" init-method="initMethod" class="dev.yavuztas.spring.context.SampleBean">
    <property name="message" value="Initial value" />
</bean>

{% endhighlight %}
As for the **last** step, we can define a *BeanPostProcessor* to run a custom operation before a spring bean initializes or after. We should pay attention that *BeanPostProcessor* classes are executed for every bean defined in our spring context.

Let's create a *BeanPostProcessor*:
{% highlight java %}
public class CustomBeanPostProcessor implements BeanPostProcessor {

	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {

		System.out.println("--- postProcessBeforeInitialization executed ---");

		return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {

		System.out.println("--- postProcessAfterInitialization executed ---");

		return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
	}

}
{% endhighlight %}
Of course, we need to register our new post-processor as a spring bean to make it work:
{% highlight xml %}

<bean class="dev.yavuztas.spring.context.CustomBeanPostProcessor" />

{% endhighlight %}
When we boot our spring context, we will see that our *CustomBeanPostProcessor*'s *postProcessBeforeInitialization* method is executed right before the *@PostConstruct* and the other method *postProcessAfterInitialization* is executed after *init-method*.

For the **destruction** stage of spring beans, we can also configure *@PreDestroy*, *DisposableBean* and *destroy-method*.  

Let's define these methods for our bean:
{% highlight java %}
public class SampleBean implements BeanNameAware, BeanFactoryAware, InitializingBean, DisposableBean {

    // rest of the class omitted

    @PreDestroy
	public void preDestroy() {

		System.out.println("--- @PreDestroy executed ---");

	}

	@Override
	public void destroy() throws Exception {

		System.out.println("--- destroy executed ---");

	}

	public void destroyMethod() {

		System.out.println("--- destroy-method executed ---");

	}

}
{% endhighlight %}
And do not forget to configure *destroy-method* in our bean config:
{% highlight xml %}

<bean id="sampleBean" init-method="initMethod" destroy-method="destroyMethod" class="dev.yavuztas.spring.context.SampleBean">
    <property name="message" value="Initial value" />
</bean>

{% endhighlight %}

**Finally**, if we close our running spring-context, we will see our destruction methods are executed in the order as;
 * *@PreDestroy*
 * *DisposableBean*
 * *destroy-method*

### Closing an *ApplicationContext*
We can close Spring's *ApplicationContext* by simply calling the method *close* of the context:
{% highlight java %}

appContext.close();

{% endhighlight %}
Besides, we do not have to manually close, instead, we can easily configure *ApplicationContext* to automatically close itself on JVM shutdown:
{% highlight java %}

appContext.registerShutdownHook();

{% endhighlight %}
Furthermore, if we are using **Spring Boot**, we do not need to do anything to close our application context because it is automatically configured to be closed in the shutdown stage.
### Conclusion
As always, you can check the code samples for this post [over on Github](https://github.com/yavuztas/java-spring-fundamentals/tree/master/src/main/java/dev/yavuztas/spring/context){:target="blank"}.

After we talked about *ApplicationContext*, in our [next post]({{ site.url }}), we will go on with the *component scanning and auto-injection* of spring beans.
