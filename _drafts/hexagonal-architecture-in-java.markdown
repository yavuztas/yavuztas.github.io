---
layout: post
title:  "A quick and practical example of Hexagonal Architecture in Java"
date:   2014-11-30 14:34:25
categories: java patterns architectures
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---
### 1. Overview
One of the most popular methods in a software application is design by layers. When modeling our application we generally use segregated layers, like the business logic layer, data access layer, etc.

There is no doubt that layered architectures bring us complexity. However, this is a fair trade-off in the benefits of **reusability**, **maintainability**, **testability** and so on. These are crucial qualities that every ideal software application should have.

Today, we will explain a special form of layered architecture which is called the **Hexagonal Architecture**. After that, we will show an example implementation of this architecture in Java.

### 2. What is the Hexagonal Architecture?
Despite not being a new pattern from bottom to top, **Hexagonal Architecture** is an improved version of **Layered Architecture**.

The main difference is separating both input and output from the central logic that handles our main business operations. This separation is not only bound to infrastructure but also includes the user interaction side.

To properly isolate the business logic and domain, this architecture mainly uses **ports** and **adapters**.

With different adapters a single port can have, we can easily define many adapters and reuse them independent from outside. This brings us the benefit of **reusability** and **testability**.

### 3. An Example Implementation in Java
Let's provide a straightforward sample to show how to implement a Hexagonal Architecture in Java. It will be a simple account application.

This application has a single domain object and a very simple operation which is creating a new record. We will use interfaces for ports and their implementations will be our adapters.

There will be three layers of our application.

### 3.1. Central Application
Central Application is the core application that contains our domain and business logic.

We have a single domain object named as *Account*:
{% highlight java %}
public class Account {

    private UUID id;

    private String name;

    private AccountType type;

    // standard getters and setters

}
{% endhighlight %}

Next, a service that is responsible for creating a new account:
{% highlight java %}
public class AccountService implements IAccountService {

    private IAccountRepository accountRepository;

    @Override
    public void createAccount(String name, String type) {

        Account account = new Account();
        account.setName(name);

        if ("user".equals(type)) {
            account.setType(AccountType.USER);
        }
        if ("mod".equals(type)) {
            account.setType(AccountType.MODERATOR);
        }

        accountRepository.save(account);

    }

}
{% endhighlight %}

Besides, it includes two interfaces as our ports.

First, *IAccountUI* interface is the port which is a contract for user interaction inputs for the domain *Account*:
{% highlight java %}
public interface IAccountUI {

    public void create(Map<String, String> parameters);

}
{% endhighlight %}

Second, *IAccountRepository* interface is another port for saving the output which is the records of *Account* domain:
{% highlight java %}
public interface IAccountRepository {

    public void save(Account account);

}
{% endhighlight %}

### 3.2. Console UI
We also have an adapter that is responsible for handling user interaction inputs.

*ConsoleUI* simply reads the user input from the console and sends it to our Central Application:
{% highlight java %}
public class ConsoleUI implements IAccountUI {

    private IAccountService accountService;

    public Map<String, String> readParameters() {

        Scanner scanner = new Scanner(System.in);

        Map<String, String> params = new HashMap<>();

        String name = scanner.next();
        String type = scanner.next();

        params.put("name", name);
        params.put("type", type);

        return params;
    }

    @Override
    public void create(Map<String, String> parameters) {

        String name = parameters.get("name");
        String type = parameters.get("type");

        accountService.createAccount(name, type);
    }

}
{% endhighlight %}

### 3.3. In Memory Account Repository
We need another adapter to handle the output of our Central Application. This output will be the account data to be saved.

*InMemoryAccountRepository* contains a basic implementation to store the *Account* data in memory:
{% highlight java %}
public class InMemoryAccountRepository implements IAccountRepository {

    private List<Account> accountList = new ArrayList<>();

    @Override
    public void save(Account account) {

        account.setId(UUID.randomUUID());

        accountList.add(account);

    }

}
{% endhighlight %}

### 3.4. Extending Our Application
As a result of Hexagonal Architecture, we may have different adapters of a single port easily. Thus this leads our application to have a great advantage of extendability.

For example, we may extend our account application by implementing a *WebUI* adapter to get user inputs from a web-based interface or a *FileAccountRepository* to persist account data into the filesystem.

### 4. Conclusion
In this tutorial, **we learned what the Hexagonal Architecture is and how to implement a quick and practical example of it in Java**.

Like always, all the code samples shown in this tutorial are available [over on GitHub](https://github.com).
