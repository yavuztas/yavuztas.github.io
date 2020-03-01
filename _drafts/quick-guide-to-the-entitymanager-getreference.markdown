---
layout: post
title:  "Quick Guide to the EntityManager getReference"
date:   2014-11-30 14:34:25
categories: java
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## 1. Introduction
--Give link to the article https://www.baeldung.com/hibernate-entitymanager
In this tutorial, we are going to reveal why it is better to use *EntityManager* *getReference* in some cases.

## 2. Fetching Entities in EntityManager
Let's see how we can fetch entities by their identity. Without writing any queries, JPA provides us two basic methods to achieve this.

### 2.1. *find()* Method
This method makes a roundtrip to underlying database, executes a select query and if the relevant record exists, it delivers us the result as an initialized entity.
### 2.2. *getReference()* Method
Similar to the *find()* method, *getReference()* also delivers the result entity. However, the returning object is an uninitialized entity which means a proxy object waiting to be initialized lazily, only if we request the fields other than entity's identity.     

## 3. An Example Use-case
An example scenario of players and game which many players have one game.

### 3.1. Domain Model
Giving models of Player and Game

### 3.2. Configuring Relations
Defining necessary fields to configure a @ManyToOne relationship whereas one game has many players.

## 4. Test Cases

### 4.1. Updating An Entity
Two test methods that updates a Game instance.
The first one uses *find()* method for getting the Game entity. Show the underlying queries when test method is executed
The second one uses *getReference()* method for getting the Game entity. Show the underlying queries when test method is executed

### 4.2. Setting up the Relation Between Entities
Two test methods that create a new Player, assign it to an existing Game and persist it.
The first one uses *find()* method when fetching the Game entity. Show the underlying queries when test method is executed
The second one uses *getReference()* method when fetching the Game entity. Show the underlying queries when test method is executed

-- NOTICE: Maybe we should add a note about we may get *EntityNotFound* exception while we are using *getReference()*. This behaivor also depends on the underlying persistence implemntation like Hibernate. In Hibernate *hibernate.jpa.compliance.proxy* property is false by default and it does not throw exception when we try to fetch the entity by *getReference* method. If we set it true then Hibernate initializes the proxy object in any case that means it fires a select query even if we use *getReference*.

## 5. Conclusion
In this tutorial, **we learned something super cool :)**.

Like always, all the code samples used in this tutorial are available [over on GitHub](https://github.com).
