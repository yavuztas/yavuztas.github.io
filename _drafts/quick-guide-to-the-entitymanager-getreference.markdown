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
First of all, we'll take a look at how we can fetch entities by their primary keys. Without writing any queries, *EntityManager* provides us two basic methods to achieve this.

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

However, the returning entity is a proxy object and it has only the primary key field initialized. The rest remains uninitialized unless we lazily request the other fields.

## 3. An Example Use-Case
In order to demonstrate *EntityManager* fetch operations, we'll create two models as our domain, *Game* and *Player*, that many players can be involved in the same game.

### 3.1. Domain Model
Let's define an entity called *Game*:
{% highlight java %}
@Entity
public class Game {

    @Id
    private Long id;

    private String name;

    // standard constructors, getters, setters

}
{% endhighlight %}

Next, we define also another entity which is *Player*:
{% highlight java %}
@Entity
public class Player {

    @Id
    private Long id;

    private String name;

    // standard constructors, getters, setters

}
{% endhighlight %}

### 3.2. Configuring Relations
In order to configure a *@ManyToOne* relation from *Player* to *Game*, let's add a *game* property to *Player* entity:
{% highlight java %}
@ManyToOne
private Game game;
{% endhighlight %}

## 4. Test Cases
Before we start our test methods, it's a good practice to define our test data seperately:
{% highlight java %}
entityManager.getTransaction().begin();

entityManager.persist(new Game(1L, "Game 1"));
entityManager.persist(new Game(2L, "Game 2"));
entityManager.persist(new Player(1L,"Player 1"));
entityManager.persist(new Player(2L, "Player 2"));
entityManager.persist(new Player(3L, "Player 3"));

entityManager.getTransaction().commit();
{% endhighlight %}
Besides, to examine underlying SQL queries, we should configure hibernate's *hibernate.show_sql* property in our *persistence.xml:*
{% highlight xml %}
<property name="hibernate.show_sql" value="true"/>
{% endhighlight %}
### 4.1. Updating Entity Fields
First, we'll check the most common way of updating an entity by using *find()* method.

So, let's write a test method to fetch the *Game* entity first, then simply update its *name* field:
{% highlight java %}
Game game1 = entityManager.find(Game.class, 1L);
game1.setName("Game Updated 1");

entityManager.persist(game1);
{% endhighlight %}
Running the test method shows us the executed SQL queries:
{% highlight java %}
Hibernate: select game0_.id as id1_0_0_, game0_.name as name2_0_0_ from Game game0_ where game0_.id=?
Hibernate: update Game set name=? where id=?
{% endhighlight %}
As we notice, the *SELECT* query looks an unnecessary roundtrip in such a case. Since we don't need to read any field of *Game* entity before our update operation, we are wondering if there is some way to execute only the *UPDATE* query.

So, let's see how the *getReference()* method behaves in the same scenario:
{% highlight java %}
Game game1 = entityManager.getReference(Game.class, 1L);
game1.setName("Game Updated 2");

entityManager.persist(game1);
{% endhighlight %}
Surprisingly, the result of the running test method is still the same and we see the *SELECT* query remains.

As we can see, Hibernate does execute a *SELECT* query whether *getReference()* is used or not when we update an entity field.

Therefore, **using *getReference()* method does not avoid the extra *SELECT* query if we execute any setter function of the reference proxy.**

### 4.2. Deleting Entities
Another similar scenario can happen when we want to execute delete operations.

Let's define another two test methods to delete a *Player* entity:
{% highlight java %}
Player player2 = entityManager.find(Player.class, 2L);
entityManager.remove(player2);
{% endhighlight %}
{% highlight java %}
Player player3 = entityManager.getReference(Player.class, 3L);
entityManager.remove(player3);
{% endhighlight %}
Running these two test methods show us the exact same queries:
{% highlight java %}
Hibernate: select
    player0_.id as id1_1_0_,
    player0_.game_id as game_id3_1_0_,
    player0_.name as name2_1_0_,
    game1_.id as id1_0_1_,
    game1_.name as name2_0_1_ from Player player0_
    left outer join Game game1_ on player0_.game_id=game1_.id
    where player0_.id=?
Hibernate: delete from Player where id=?
{% endhighlight %}
Similarly, for delete operations, the result is identical. Even if we don't read any fields of the *Player* entity, Hibernate executes an additional *SELECT* query as well.

Hence, **there is no difference whether we choose *getReference()* or *find()* method when we need to delete an existing entity.**

### 4.3. Updating Entity Relations
Another common use-case shows up when we need to save relations between our entities.

Let's add another method to demonstrate a *Player*'s participation to a *Game* by simply updating the *Player*'s *game* property:
{% highlight java %}
Game game1 = entityManager.find(Game.class, 1L);

Player player1 = entityManager.find(Player.class, 1L);
player1.setGame(game1);

entityManager.persist(player1);
{% endhighlight %}
Running the test gives us a similar result one more time:
{% highlight java %}
Hibernate: select game0_.id as id1_0_0_, game0_.name as name2_0_0_ from Game game0_ where game0_.id=?
Hibernate: select
    player0_.id as id1_1_0_,
    player0_.game_id as game_id3_1_0_,
    player0_.name as name2_1_0_,
    game1_.id as id1_0_1_,
    game1_.name as name2_0_1_ from Player player0_
    left outer join Game game1_ on player0_.game_id=game1_.id
    where player0_.id=?
Hibernate: update Player set game_id=?, name=? where id=?
{% endhighlight %}
Now, let's define one more test to see how the *getReference()* method behaves:
{% highlight java %}
Game game2 = entityManager.getReference(Game.class, 2L);

Player player1 = entityManager.find(Player.class, 1L);
player1.setGame(game2);

entityManager.persist(player1);
{% endhighlight %}
However, running the test shows a different behavior:
{% highlight java %}
Hibernate: select
    player0_.id as id1_1_0_,
    player0_.game_id as game_id3_1_0_,
    player0_.name as name2_1_0_,
    game1_.id as id1_0_1_,
    game1_.name as name2_0_1_ from Player player0_
    left outer join Game game1_ on player0_.game_id=game1_.id
    where player0_.id=?
Hibernate: update Player set game_id=?, name=? where id=?
{% endhighlight %}
As we should notice, Hibernate doesn't execute a *SELECT* query for the *Game* entity when we use *getReference()* this time.

Finally, it looks like a good practice to go with *getReference()* method in this case. Because a proxy *Game* object is enough to set our relations without initializing its fields.

Consequently, **using *getReference()* method can avoid an unnecessary roundtrip to our database when we update entity relations.**

## 5. Hibernate First Level Cache
It can be confusing sometimes that both methods *find()* and *getReference()* may not execute any *SELECT* queries.

Let's imagine a situation that our entities already existed in the persistence context before any update operation:
{% highlight java %}
entityManager.getTransaction().begin();
entityManager.persist(new Game(1L, "Game 1"));
entityManager.persist(new Player(1L, "Player 1"));
entityManager.getTransaction().commit();

entityManager.getTransaction().begin();
Game game1 = entityManager.getReference(Game.class, 1L);

Player player1 = entityManager.find(Player.class, 1L);
player1.setGame(game1);

entityManager.persist(player1);
entityManager.getTransaction().commit();
{% endhighlight %}
Running the test results us there is only an update query:
{% highlight java %}
Hibernate: update Player set game_id=?, name=? where id=?
{% endhighlight %}
We should notice that we don't see any *SELECT* queries no matter we use *find()* or *getReference()*. In such a case, this is because our entity cached in Hibernate's *First Level Cache.*

As a result, **when our entities stored in Hibernate's First Level Cache, then both *find()* and *getReference()* methods do not hit our database to save a roundtrip.**

## 6. Different JPA Implementations
As a final reminder, we should be aware that the behavior of *getReference()* method depends on the underlying persistence provider. According to [JPA 2 Specification](https://github.com/eclipse-ee4j/jpa-api/blob/ffe76306a2846df4849612af20ef03ecc6ff4843/api/src/main/java/javax/persistence/EntityManager.java#L258), the persistence provider is allowed to throw the *EntityNotFoundException* when the *getReference()* method is called.

Therefore, it may be different for other persistence providers and we may encounter *EntityNotFoundException* when we use *getReference()*.

Nevertheless, **Hibernate doesn't follow the specification for *getReference()* by default to save a database roundtrip if possible**. So, it does not throw an exception when we retrieve reference proxies even if they do not exist in the database.

Instead, **Hibernate provides a configuration property to offer an opinionated way for those who want to follow the JPA specification.**

In such a case, we may consider setting the *hibernate.jpa.compliance.proxy* property to *true:*
{% highlight xml %}
<property name="hibernate.jpa.compliance.proxy" value="true"/>
{% endhighlight %}
Thus, Hibernate initializes the proxy object in any case which means it executes a *SELECT* query when we also use *getReference()*.

## 7. Conclusion
In this tutorial, we explored some use-cases that we can benefit from the reference proxy objects and learned how to use *EntityManager getReference()* method in Hibernate.

Like always, all the code samples and more test cases for this tutorial are available [over on GitHub](https://github.com).
