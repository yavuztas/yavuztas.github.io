
### STUDY NOTES

1. In Spring xml bean configuration if **no id is given** then Spring generates an id with **fully qualified name and a number** like *com.sample.bean.SampleBean#0*

2. About resource paths:
{% highlight java %}
ApplicationContext context = new ClassPathXmlApplicationContext("classpath:/com.example.myapp.config.xml");
{% endhighlight %}

 * The *classpath:* prefix could be omitted
 * Package name using the dot character is not well formatted, should be "/"
 * The slash character preceding com.example could be omit 

3. *Destroy* methods of **prototype** beans are **never called**

4. Lifecycle of Spring Beans:
 * *Aware*
 * *@PostConstruct*
 * *InitializingBean* afterPropertiesSet()
 * *init-method*
 * *BeanPostProcessor*
For destruction phase:
 * *@PreDestroy*
 * *DisposableBean* destroy()
 * *destroy-method*

5. For configuration classes; **default or no-arg constructor is mandatory**.
Here, the provided constructor with a dataSource parameter is not taken into account:
{% highlight java %}
public class ApplicationConfig {

    private DataSource dataSource;
    @Autowired
    public ApplicationConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean(name="clientRepository")
    ClientRepository jpaClientRepository() {
        return new JpaClientRepository();
    }

}
{% endhighlight %}

6. For xml configuration;
 * Use *<tx:annotation-driven />* to enable *@Transactional* annotation scanning
 * Use *<aop:aspectj-autoproxy />* to enable detection of *@Aspect* bean
 * Turns on *<context:annotation-config />* or *<context:component-scan />* to enable *@Autowiring* annotation
 * Turns on *<context:component-scan />* to enable *@Component* annotation scanning

7. Advantages of Spring on integration tests:
 * Spring context is mandatory. DI can be used
 * No mocking implementation or abstraction supported

8. Advantages of Spring on unit tests:
 * No need for Spring context so DI also.
 * Some mock objects provided for servlets

9. About aop pointcuts be careful; expression could not satisfied both first and second execution point. Do not confuse the && operator and || operator:
{% highlight java %}
execution(* *..AccountServiceImpl.update(..)) && execution(* * ..ClientServiceImpl.update(..))
{% endhighlight %}

10. About Spring AOP:
Due to the **proxy-based nature of Spring's AOP framework**, protected methods are by definition not intercepted, neither for JDK proxies nor for CGLIB proxies. As a consequence, any given pointcut will be matched against **public methods only!**
To intercept private and protected methods, **AspecJ weaving** should be used instead of the Spring’s proxy-bases AOP framework.

11. About this pointcut:
{% highlight java %}
execution(* com.test.service..*.*( * ))
{% endhighlight %}
 * *void com.test.service.MyServiceImpl#transfert(Money amount)* matches
 * The pattern ( * ) matches a method taking only one parameter of any type
 * The com.test.service.account sub-package matches the pointcut

12. About pointcut:
{% highlight java %}
execution(public * * (..))
{% endhighlight %}
 * Matches all public methods
 * All return types including void with all packages and methods
 * 0, 1 or many parameters

13. About read-only marked transactions:
 * May be improve performance when using Hibernate
 * Spring does not make any optimizations to its transaction interceptor
 * Provides safeguards with Oracle and some other databases

14. Some **NoSQL** databases are supported through the Spring Data project **not in Spring Framework**

15. A JdbcTemplate requires a **DataSource as input parameters**. And can open/close this datasource connection.

16. About JdbcTemplate:
 * *RowMapper*: result set parsing when needed to map each row into a custom object
 * *RowCallbackHandler*: result set parsing without returning a result to the JdbcTemplate caller
 * *ResultSetExtractor*: for result set parsing and merging all rows into a single object

17. About transaction:
 * *<tx:jta-transaction-manager />*
 * Id of the transaction manager bean could be customized (ie. txManager)
 * *DataSourceTransactionManager* is a transaction manager for a JDBC data source.
 * *HibernateTransactionManager* may be used to manage transaction with Hibernate.
 * *AbstractPlatformTransactionManager* has a *defaultTimeout* property that could be customized

18. About transaction:
{% highlight java %}

@Transactional(propagation = Propagation.REQUIRED)
public void transferMoney(Account src, Account target, double amount) {
    add(src, -amount);
    add(src, amount);
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void add(Account account, Double amount) {
    // IMPLEMENTATION
}

{% endhighlight %}

In proxy mode, only external method calls coming in through the proxy are intercepted. In the code snippet, the add() method is **self-invocated.** This means that, the *@Transactional* annotation of the add() method is not interpreted. The *REQUIRES_NEW* propagation level **is not taken into account**.

To summary, when the transferMoney() methods calls add() method directly, the transaction attributes of add() method are not used.

19. About default **rollback** policy:
The default policy for transaction rollback is **Rollback for RuntimeException**

20. About Spring MVC controller method returns:
 * Spring **does not allow to return an absolute path** to the view
 * Controller could return a String that matches with a logical view name
 * A JstlView with the .jsp path (i.e. /WEB-INF/accountList.jsp)
 * void forward to the default view
 * null forward to the default view

21. About Spring Security:
 * *@Secured* annotation is a Spring Security annotation
 * *@RolesAllowed* is a JSR-250 annotation that is supported by Spring Security
 * Spring Security could be configured in a XML way to intercept particular URLs

22. What is right about Spring Security configuration and the security namespace?

 * You cannot mix EL and constant in the same configuration file
 * If more than one intercept-url matches, the top one is used
 * Ant pattern is used by default. But you can change to use regular expression.
 * Security rules may apply to request URL, request method (GET, POST ...) but not to request parameters.

23. The HTTP 201 status code means "Resource created”. It follows a POST command this indicates success.

24. Which of the following statements is true regarding the *@ResponseStatus* annotation?
 * Starting from Spring Framework 4.2, the *@ResponseStatus* annotation is detected on nested exceptions.
 * The *ResponseStatusExceptionResolver* uses the *@ResponseStatus* annotation to map exception to HTTP status code.
 * The response status can be set by RedirectView but a controller handler is annotated with the *@ResponseStatus* **takes precedence** over the this value.
 * *@ResponseStatus* annotation on a *@RestController* class is not supported

25. Compared to monolithic application, what are the properties of microservices?
 * The base code is easy to understand
 * Easier deployment
 * Fine-grained scaling
 * **Con:** Complex development, hard to program

26. What Spring Cloud provides in a microservices architecture?

 * Spring Cloud supports Service Discovery solution as **Eureka** and **Consul**. **But it does not implement the Service Discovery pattern.**
 * The Spring Cloud Config project provides both a server and a client-side support for externalized configuration in a distributed system.
 * Spring Cloud **does not support Docker out of the box**
 * Spring Cloud supports Netflix implementation of common microservices patterns:
 Service Discovery (Eureka), Circuit Breaker (Hystrix), Intelligent Routing (Zuul) and Client Side Load Balancing (Ribbon).

 27. Creating a deployable .war file in Spring Boot:
 When creating a 'war' file which will be deployed into a server, it is best to add the 'provided' setting below, to avoid the server files being included in the 'war'.

	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-tomcat</artifactId>
		<scope>**provided**</scope>
	</dependency>

If you use Maven and spring-boot-starter-parent, you must change the pom.xml 'packaging' setting to below.

    <packaging>war</packaging>

The first step in producing a deployable war file is to provide a *SpringBootServletInitializer* subclass.

28. About JPA configuration:
 * You can set up an *EntityManager* using *LocalEntityManagerFactoryBean*.
 * You can set up an *EntityManager* using *LocalContainerEntityManagerFactoryBean*.
 * The *persistence.xml* is not required.  The *LocalContainerEntityManagerFactoryBean* supports a 'packagesToScan' property where the packages to scan for *@Entity* classes can be specified.
 * To enable JPA in a Spring Boot application, add the *spring-boot-starter* and *spring-boot-starter-data-jpa* dependencies.

29. About Spring MVC resolvers, beware they are all interfaces!
 * *interface HandlerExceptionResolver*:  Maps exceptions to views also allowing for more complex exception handling code.
 * *interface ViewResolver*:  Resolves logical String-based view names to actual View types.
 * *interface ThemeResolver*:  Resolves themes your web application can use, for example, to offer personalized layouts

30. Which of these is a way of closing an application context, that allows you to listen to an event, so that you can do some additional actions when the application context is closing?

These 2 ways below will result in the publishing of a *ContextClosedEvent*.  You can create a listener to listen to the mentioned event, and do some actions when the application context is closing.
 * Call *close()* on the application context
 * Use the *registerShutdownHook()* on the application context

Both of them will result in *doClose()* being called.  *doClose()* publishes the *ContextClosedEvent*.

31. Some features of Spring AOP:
 * can use CGLIB Proxies
 * can introduce new interfaces (and a corresponding implementation) to an advised object
 * can proxy objects that does not implement an interface
 * can weave the aspects in **only runtime, not in compile time**

32. About *BeanPostProcessor* and *BeanFactoryPostProcessor*:
 * Both *BeanPostProcessor* and *BeanFactoryPostProcessor*, can control the order in which they execute by implementing the *Ordered* or *PriorityOrdered* interface, not just *BeanFactoryPostProcessor*.
 * *BeanPostProcessor* (not BeanFactoryPostProcessor) operates on bean (or object) instances; that is, it does its work after the container has instantiated a bean instance.
 * *BeanFactoryPostProcessor* operates on the **bean configuration metadata**; that is, the Spring IoC container allows a *BeanFactoryPostProcessor* to read the configuration metadata and potentially change it before the container instantiates any beans other than *BeanFactoryPostProcessors*.
 * The *BeanFactoryPostProcessor* does its work **BEFORE** the container instantiates any beans. The *BeanPostProcessor* does its work **AFTER** the container has instantiated a bean instance.

33. Configuring a *DataSource* in Spring:

    DriverManagerDataSource myDataSource = new DriverManagerDataSource();
    myDataSource.setDriverClassName(.....);
    myDataSource.setUrl(.....);
    myDataSource.setUsername(.....);
    myDataSource.setPassword(.....);

34. About database system isolation levels:

    Read uncommitted
        dirty read can occur:  Yes
        non-repeatable read can occur:  Yes
        phantom read can occur:  Yes

    Read committed
        dirty read can occur:  No
        non-repeatable read can occur:  Yes
        phantom read can occur:  Yes

    Repeatable reads
        dirty read can occur:  No
        non-repeatable read can occur:  No
        phantom read can occur:  Yes

    Serializable
        dirty read can occur:  No
        non-repeatable read can occur:  No
        phantom read can occur:  No

35. Spring Security jsp tag library:
 * *authorize*: whether its content should be evaluated or not.
 * *authentication*: tag allows to access *Authentication* object stored in the *SecurityContext*.
 * *accesscontrollist*: tag only valid with Spring Security's ACL module

36. Places for Spring Boot refers to property sources:
 * *SpringApplication.setDefaultProperties()*
 * JNDI attributes
 * application.properties or .yml files inside or outside of the jar package.
 * OS environment variables

application-properties.xml **cannot be taken** from outside of the jar package.

37. About Spring Security xml configuration:
"'filters' can only take the value "none".  This will cause any matching request to bypass the Spring Security filter chain entirely.  None of the rest of the <http> configuration will have any effect on the request and there will be no security context available for its duration. Access to secured methods during the request will fail."

Below is an example.

    <intercept-url pattern="/** " access="ROLE_USER" filters="none" ></intercept>
    <http pattern="/css/** " security="none"></http>

The Spring Security Reference describes the 'security' attribute as:
"A request pattern can be mapped to an empty filter chain, by setting this attribute to none. No security will be applied and none of Spring Security's features will be available."

38. *PropertySourcesPlaceholderConfigurer* class is an implementation of *PlaceholderConfigurerSupport*. Also *@Autowired* annotations **cannot be resolved** inside of it. However, *@Value* annotations can be resolved.

39. An example of proper ordering of Spring Security filters:

BasicAuthenticationFilter, AnonymousAuthenticationFilter, FilterSecurityInterceptor

40. Application servers manages *global transactions* JTA. Another way of using this transactions in the past was *EJB CMT (Container Managed Transactions)*

41. *@Qualifier* annotation can be used on **field, type, parameter and methods.**

42. *BeanNameUrlHandlerMapping* is included in the Spring Framework, and it is the default if there is no *HandlerMapping* bean registered in the application context.

43. For *JdbcTemplate* methods we have:

 * *query* returns *List*.
 * *queryForMap* returns *Map*.
 * *queryForObject* returns *Object* or *Object[]*.
 * *queryForList* return *List*.

44. About Spring Transactions, you can call *setRollbackOnly()* on the *TransactionStatus* object to roll back the current transaction.

45. Spring *@Lookup* method injection:
{% highlight java %}
@Component
public class MySingletonBean {

    public void showMessage(){
        MyPrototypeBean bean = getPrototypeBean();
        //each time getPrototypeBean() call
        //will return new instance
    }

    @Lookup
    public MyPrototypeBean getPrototypeBean(){
        //spring will override this method
        return null;
    }
}
{% endhighlight %}

Spring uses CGLIB to proxy these lookup methods. It would be something like this:
{% highlight java %}

public MyPrototypeBean getPrototypeBean(){
    return applicationContext.getBean(MyPrototypeBean.class);
}

{% endhighlight %}

46. About Spring Security *antMatchers* and *mvcMatchers*:

* *antMatchers("/secured")* matches only the exact **/secured** URL.
* *mvcMatchers("/secured")* matches **/secured** as well as **/secured/**, **/secured.html**, **/secured.xyz**.
