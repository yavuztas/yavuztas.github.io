---
layout: post
title:  "Spring Ultimate Guide"
date:   2019-06-15 14:52:00
categories: java spring
tags: featured
image: "/assets/images/header-spring-3.png"
image2: "/assets/images/header-spring-mobile.png"
---
### What is dependency injection and what are the advantages?
**Dependency Injection** is a popular design pattern to implement IoC Containers. It is a mechanism that scans object hierarchy, resolves dependencies and assigns the instances properly to dependent objects.

For example, in **Spring Framework** this is clearly provided by the well-known annotation *@Autowired*. Spring's *ApplicationContext* makes a component scan on startup and handles the injection of dependent instances internally among its managed objects which are called as **Spring Beans**.

The advantages of Dependency Injection are:
* high cohesion
* loosely coupled objects
* maintainability
* reusability
* testability

### What is an interface and what are the advantages of making use of them in Java?
Using interfaces instead of concrete classes definitely has great benefits on implementing OOP's solid principles such as; **Open Closed Principle**, **Dependency Inversion Principle**. This method is also so known as **programming to interface**. Besides with the help of interfaces, some critical benefits can be achievable like **reusability**, **testability**.

In **Spring Framework**, interfaces are advised for spring beans. It makes it possible to provide **inversion of control** and **testability**.

### What is meant by *application-context*?
**ApplicationContext** is the main container that exposes **BeanFactory**. *BeanFactory* is the basic IoC container of Spring. *ApplicationContext* is the central container that provides many ready to use business features such as; *MessageSource* properties, event publishing, pre/post processors, etc. for enterprise-level applications. In addition, *ApplicationContext* provides direct access to *BeanFactory*.

### How are you going to create a new instance of an ApplicationContext?
There are pretty much *ApplicationContext* types but all of them can be initialized with a simple object creation call:
>FileSystemXmlApplicationContext
{:.filename}

{% highlight java %}
FileSystemXmlApplicationContext appContext = new FileSystemXmlApplicationContext("src/main/resources/beans.xml");
{% endhighlight %}
>ClassPathXmlApplicationContext
{:.filename}

{% highlight java %}
ClassPathXmlApplicationContext appContext = new ClassPathXmlApplicationContext("beans.xml");
{% endhighlight %}
>AnnotationConfigApplicationContext
{:.filename}

{% highlight java %}
AnnotationConfigApplicationContext appContext = new AnnotationConfigApplicationContext("com.yavuztas.spring.beans");
{% endhighlight %}

### Can you describe the lifecycle of a Spring Bean in an ApplicationContext?
The lifecycle of spring beans can be grouped into these sections:
* Aware interfaces for injecting infrastructure components (BeanNameAware, BeanFactoryAware, etc.)
* *@PostConstruct* and *@PreDestroy* annotations
* *InitializingBean* and *DisposableBean* for callback execution
* *init-method* and *destroy-method* configurations
* *BeanPostProcessor* for altering beans before or after initialization

For the destruction phase these are executed in order:
* *@PreDestroy*
* *DisposableBean*
* *destroy-method*

### How are you going to create an ApplicationContext in an integration test?
You can implement *ApplicationContextAware* interface or directly inject it with *@Autowired*.

### What is the preferred way to close an application context? Does Spring Boot do this for you?
{% highlight java %}
// you can register a jvm shutdown hook
appContext.registerShutdownHook();
// or directly close it also this is very rare!
appContext.close();
{% endhighlight %}
If you are already using **Spring Boot** then it is closed automatically.

### Dependency injection using Java configuration?
{% highlight java %}
@Configuration
public class AppConfig {

    @Bean
    public UserService getUserService(UserDao userDao) {
        return new UserServiceImpl(userDao);
    }

    @Bean
    public UserDao getUserDao() {
        return new UserDaoImpl();
    }
}
{% endhighlight %}

### Dependency injection using annotations
{% highlight java %}
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;

    // other business logic methods
}
{% endhighlight %}

### Component scanning, Stereotypes?
Component scan can be enabled by *context:component-scan*:
{% highlight xml %}
<context:component-scan base-package="dev.yavuztas.spring.service" />
<context:component-scan base-package="dev.yavuztas.spring.dao" />
<context:component-scan base-package="dev.yavuztas.spring.controller" />
{% endhighlight %}

You do not need to define *context:annotation-config* anymore if you use *context:component-scan*

**@Component**: Spring basic marker annotation in order to tell spring to find and pick up as bean if automatic component scan is enabled.

**@Repository**: Besides being a marker for beans like *@Component*, it also makes the unchecked exceptions (thrown from DAO methods) eligible for translation into Spring *DataAccessException*.

**@Service**: No additional behavior besides marking a class as a bean. However it shows the intent of the seperation of layers for service classes.

**@Controller**: The annotation marks a class as a Spring Web MVC controller

### Scopes for Spring beans? What is the default scope?
**singleton (default)**: Single bean object instance per spring IoC container

**prototype**: Opposite to singleton, it produces a new instance each and every time a bean is requested.

**request**: A single instance will be created and available during complete lifecycle of an HTTP request. Only valid in web-aware Spring ApplicationContext.

**session**: A single instance will be created and available during complete lifecycle of an HTTP Session. Only valid in web-aware Spring ApplicationContext.

**application**: A single instance will be created and available during complete lifecycle of ServletContext. Only valid in web-aware Spring ApplicationContext.

**websocket**: A single instance will be created and available during complete lifecycle of WebSocket. Only valid in web-aware Spring ApplicationContext.

### Are beans lazily or eagerly instantiated by default? How do you alter this behavior?
All bean initialized **eager** by default. You can alter this behavior by *@Lazy* annotation. Or with the property *lazy=true* in XML configuration.

### What is a property source? How would you use @PropertySource?
Property source is an external file source that contains properties as key/value pairs.
{% highlight java %}
@Configuration
@ComponentScan(basePackages = { "dev.yavuztas.&ast;" })
@PropertySource("classpath:config.properties")
public class AppConfig {

	@Value("${mongodb.url}")
	private String mongoUrl;

	@Value("${mongodb.db}")
	private String mongoDb;

    @Autowired
	private Environment env;

	// To resolve ${} in @Value we need to define PropertySourcesPlaceholderConfigurer
	@Bean
	public static PropertySourcesPlaceholderConfigurer propertyConfigInDev() {
		return new PropertySourcesPlaceholderConfigurer();
	}

    @Bean
	public MongoTemplate mongoTemplate() throws Exception {

        // recommended property access by Environment, if so we do not need to define PropertySourcesPlaceholderConfigurer
		String mongodbUrl = env.getProperty("mongodb.url");
		String defaultDb = env.getProperty("mongodb.db");

		// initialize MongoTemplate
		return new MongoTemplate();

	}

}
{% endhighlight %}

### What is a *BeanFactoryPostProcessor* and what is it used for? When is it invoked?
*BeanFactoryPostProcessor* allows for custom modification of an application context's bean definitions (not instances), adapting the bean property values of the context's underlying bean factory. *BeanFactoryPostProcessor* instances are automatically detected by context and executed before all beans created.

*PropertyResourceConfigurer* and its concerete implementations are good examples of these use cases.

### Why would you define a static @Bean method?
If you need to instantiate a bean before all the other beans created you need to define it as static. *PropertySourcesPlaceholderConfigurer* in Spring is a good example of this use case.

### What is a *ProperySourcesPlaceholderConfigurer* used for?
*ProperySourcesPlaceholderConfigurer* used for resolving ${...} placeholders for @Value annotations.

### What is a *BeanPostProcessor* and how is it different to a *BeanFactoryPostProcessor*?
*BeanPostProcessor* is for altering the bean instances on the other hand *BeanFactoryPostProcessor* is for altering bean definitions before they are created.

### What is an initialization method and how is it declared on a Spring bean?
In the XML configuration you can define for a specific bean as *init-method* and *destroyMethod*:
{% highlight xml %}
<beans>

	<bean id="sampleBean" class="dev.yavuztas.spring.context.SampleBean" init-method="initMethod" destroy-method="destroyMethod"></bean>

</beans>
{% endhighlight %}
Or you can define them globally for all beans as *default-init-method* and *default-destroy-method*:
{% highlight xml %}
<beans default-init-method="initMethod" default-destroy-method="destroyMethod">

	<bean id="sampleBean" class="dev.yavuztas.spring.context.SampleBean"></bean>

</beans>
{% endhighlight %}

### Consider how you enable JSR-250 annotations like @PostConstruct and @PreDestroy? When/how will they get called?
In order to make these annotations work you need to define *CommonAnnotationBeanPostProcessor* as a bean. They are called before init and destroy methods, right before *afterPropertiesSet* of *InitializingBean* and *destroy* of *DisposableBean* interfaces.

### How else can you define an initialization or destruction method for a Spring bean?
By using *afterPropertiesSet* method of *InitializingBean* and *destroy* method of *DisposableBean* interfaces.

### What is the behavior of the annotation @Autowired with regards to field injection, constructor injection and method injection?
You simply do not have to define it in XML so it will be pretty much dynamic and hustle-free of managing configurations.

### What do you have to do, if you would like to inject something into a private field? How does this impact testing?
You can normally inject any private field because Spring uses reflection to set these values implicitly. However when it comes to testing, you may not easily mock any private field and this is where the constructor and setter injection comes in handy on some occassions.

### How does the @Qualifier annotation complement the use of @Autowired?
When you have more than one bean instance of same type you can define an alias for each instance by the help of @Qualifier annotation.

### What is a proxy object and what are the two different types of proxies Spring can create?
To be continued...
