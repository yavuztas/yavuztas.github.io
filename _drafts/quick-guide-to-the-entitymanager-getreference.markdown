---
layout: post
title:  "Quick Guide to the EntityManager getReference()"
date:   2014-11-30 14:34:25
categories: java
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## 1. Introduction
*EntityManager getReference()* method has been a part of the JPA specification since the first version. However, this method confuses some developers because its behavior varies depending on the underlying persistence provider.

In this tutorial, we are going to reveal how to use *getReference()* method in [*Hibernate EntityManager*](https://www.baeldung.com/hibernate-entitymanager).

## 2. *EntityManager* Fetch Operations
First of all, we'll take a look how we can fetch entities by their primary keys. Without writing any queries, *EntityManager* provides us two basic methods to achieve this.

### 2.1. *find()* Method
*find()* is the most common method of fetching entities:
{% highlight java %}
Game game = entityManager.find(Game.class, 1L);
{% endhighlight %}

However, this method initializes the entity in the first place as we request.

### 2.2. *getReference()* Method
Similar to the *find()* method, *getReference()* is also another way to retrieve entities:
{% highlight java %}
Game game = entityManager.getReference(Game.class, 1L);
{% endhighlight %}

However, the returning entity is a proxy object that only the primary key field initialized. It remains uninitialized unless we lazily request the other fields.

## 3. An Example Use-Case
In order to demonstrate *EntityManager* fetch operations, we'll create two models as our domain, *Game* and *Player*, that many players can be involved in the same game.

### 3.1. Domain Model
Let's define an entity called *Game*:
{% highlight java %}

{% endhighlight %}

Next, we define also another entity which is *Player*:
{% highlight java %}

{% endhighlight %}

### 3.2. Configuring Relations
In order to set up the relation between *Player* and *Game* entities, let's configure a *@ManyToOne* association from *Player* to *Game*:
{% highlight java %}

{% endhighlight %}

## 4. Test Cases
Before we start our test methods, it's a good practice to define our test data seperately:
{% highlight java %}
@Before...
{% endhighlight %}
Besides, to examine underlying sql queries, we should configure hibernate's *hibernate.show_sql* property in our *persistence.xml:*
{% highlight xml %}
<property name="hibernate.show_sql" value="true"/>
{% endhighlight %}
### 4.1. Updating Entities
First, we'll check the most common way of updating an entity by using *find()* method.

So, let's write a test method to fetch the *Game* entity first, then simply update its *name* field:
{% highlight java %}
@Test
public void whenUsingFindMethodToUpdateGame_thenItShouldFireSelectQuery(){

}
{% endhighlight %}
Running the test method shows us the executed SQL queries:
{% highlight java %}
Select ...
Update ...
{% endhighlight %}
As we notice, the *SELECT* query looks an unnecessary roundtrip in such a case. Since we don't need to read any field of *Game* entity before our update operation, there should be some proper way to execute only the *UPDATE* query.

So, let's see how the *getReference()* method behaves in the same scenario:
{% highlight java %}
@Test
public void whenUsingGetReferenceMethodToUpdateGame_thenItShouldNotFireSelectQuery(){

}
{% endhighlight %}
Therefore, the result of the running test method will be without a *SELECT* query:
{% highlight java %}
Update ...
{% endhighlight %}
As we can see, Hibernate doesn't execute any *SELECT* query when *getReference()* is used. But instead, it returns only a proxy object with the primary key field.

Certainly, **using *getReference()* method can avoid an unnecessary roundtrip to our database by saving an extra *SELECT* query.**

### 4.2. Updating Entity Relations
Another common use-case shows up when we need to save relations between our entities.

Let's implement another method to demonstrate a *Player*'s participation to a *Game* by simply updating the *Player*'s *game* property:
{% highlight java %}
@Test
public void whenUsingGetReferenceMethodToUpdatePlayersGame_thenItShouldNotFireSelectQuery(){

}
{% endhighlight %}
Similarly, unless we need to read any fields of *Game* entity, it's a good practice to go with *getReference()* method because only a proxy *Game* object is enough to set our relations.

Consequently, **using *getReference()* method can be an efficient way when we only need to update entity relations.**

### 4.3. Deleting Entities
Another similar scenario can be happen when we want to execute delete operations.

Let's define one more test method to delete a *Player* entity:
{% highlight java %}
@Test
public void whenUsingGetReferenceMethodToDeletePlayer_thenItShouldNotFireSelectQuery(){

}
{% endhighlight %}
Certainly for delete operations, if we don't have any specific requirements we might not need additional *SELECT* query as well.

Thus, **it can be better to choose *getReference()* when we only need to delete an existing entity.**

## 5. Different JPA Implementations
As a final reminder, we should be aware that the behavior of *getReference()* method depends on the underlying persistence provider. According to [JPA 2 Specification](https://github.com/eclipse-ee4j/jpa-api/blob/ffe76306a2846df4849612af20ef03ecc6ff4843/api/src/main/java/javax/persistence/EntityManager.java#L258), the persistence provider is allowed to throw the *EntityNotFoundException* when the *getReference()* method is called.

Therefore, it may be different for other persistence providers and we may encounter *EntityNotFoundException* when we use *getReference()*.

However, Hibernate doesn't follow the specification to save a database roundtrip. So, it does not throw an exception when we retrieve reference proxies even if they do not exist in the database.

Instead, **Hibernate provides a configuration property to offer an opiniated way for those who want to follow the JPA specification.**

In such a case, we may consider setting the *hibernate.jpa.compliance.proxy* property to *true:*
{% highlight xml %}
<property name="hibernate.jpa.compliance.proxy" value="true"/>
{% endhighlight %}
Thus, Hibernate initializes the proxy object in any case which means it executes a *SELECT* query when we also use *getReference()*.

## 6. Conclusion
In this tutorial, we explored some use-cases that we can benefit the reference proxy objects and learned how to use *EntityManager getReference()* method in Hibernate.

Like always, all the code samples and more test cases for this tutorial are available [over on GitHub](https://github.com).
