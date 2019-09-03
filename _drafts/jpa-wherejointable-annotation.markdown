---
layout: post
title:  "JPA @WhereJoinTable Annotation"
date:   2019-10-30 14:00:00
categories: java jpa
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## 1. Overview
When we use an **ORM framework** in today's complex business requirements, we need to define **one-to-many** and **many-to-many** associations to reflect our business domain properly.

However, most of the time we may need to apply filters when we fetch entities and their *one-to-many* or *many-to-many* relations. Beyond that, **sometimes our requirement might be to acquire related entities with a given filter applied, not to a property of entities but to a property of the relation itself.**

Hopefully, **JPA specification** supports many useful and practical annotations out of the box.

In this article, we explain where and how we can use **@WhereJoinTable** annotation in **JPA** to come up with a proper solution to this requirement.

## 2. Domain Model
Let's imagine we have two simple entities, *User* and *Group* classes, which are associated as *many-to-many* to each other:
{% highlight java %}
@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @ManyToMany
    private List<Group> groups = new ArrayList<>();

    // standard getters and setters

}
{% endhighlight %}
{% highlight java %}
@Entity
public class Group {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @ManyToMany(mappedBy = "groups")
    private List<User> users = new ArrayList<>();

    // standard getters and setters

}
{% endhighlight %}
As it is defined, our *User* entity can be a member of more than one *Group* entities. Vice versa, a *Group* entity can contain more than one *User* entities.

## 3. Relation Entity
When it comes to the database side, to implement *many-to-many* associations there is a separate table which we can call as **relation table.** The relation table consists of only two columns which are the primary keys of the entity tables. Normally, our mapping definition is enough to cover this kind of relation tables.

However, when we need to map additional data over on the relation table we may also define an entity for the user-group relation itself.

Let's create *UserGroupRelation* class as our **relation entity:**
{% highlight java %}
@Entity(name = "r_user_group")
public class UserGroupRelation implements Serializable {

    @Id
    @Column(name = "user_id", insertable = false, updatable = false)
    private Long userId;

    @Id
    @Column(name = "group_id", insertable = false, updatable = false)
    private Long groupId;

}
{% endhighlight %}
Let's say we want to store every *User*'s role for each *Group*. So, we create *UserGroupRole* enumeration:
{% highlight java %}
public enum UserGroupRole {
    MEMBER, MODERATOR
}
{% endhighlight %}
Next, we add a *role* property to *UserGroupRelation*:
{% highlight java %}
@Enumerated(EnumType.STRING)
private UserGroupRole role;
{% endhighlight %}

Depending on the requirement, we should notice that **we do not always need to define a relation entity.**

Especially for the applications which their database are populated from outside and the corresponding relation table already exists. In this case, we may simply omit to create an entity.

Whether we have a relation entity or not, **we need to add *@JoinTable* annotation on *User*'s *groups* collection**. Thus, we specify the join table name to point out to the same one of *UserGroupRelation* entity or our existing relation table:
{% highlight java %}
@ManyToMany
@JoinTable(
    name = "r_user_group",
    joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "group_id")
)
private List<Group> groups = new ArrayList<>();
{% endhighlight %}

## 4. Sample Data
For our integration tests, let's define our sample data:
{% highlight java %}
public void setup() {

    user1 = new User("user1");
    user2 = new User("user2");
    user3 = new User("user3");

    group1 = new Group("group1");
    group2 = new Group("group2");

    groupRepository.save(group1);
    groupRepository.save(group2);

    userRepository.save(user1);
    userRepository.save(user2);
    userRepository.save(user3);

    saveRelation(user1, group1, UserGroupRole.MODERATOR);
    saveRelation(user2, group1, UserGroupRole.MODERATOR);
    saveRelation(user3, group1, UserGroupRole.MEMBER);

    saveRelation(user1, group2, UserGroupRole.MEMBER);
    saveRelation(user2, group2, UserGroupRole.MODERATOR);

}

public void saveRelation(User user, Group group, UserGroupRole role) {

    UserGroupRelation relation = new UserGroupRelation(user.getId(), group.getId(), role);
    entityManager.persist(relation);
    entityManager.flush();
    entityManager.refresh(user);
    entityManager.refresh(group);

}
{% endhighlight %}
As we can see, *user1* and *user2* each is in two groups. Also, we should notice that while *user1* is *MODERATOR* on *group1*, at the same time it has a *MEMBER* role on *group2*.

After we defined our sample data, we will cover how to fetch groups of a user in the next section.

## 5. Fetching *@ManyToMany* Relations
In order to fetch groups of a user, we can simply invoke the *getGroups()* method of *User* inside an active JPA session:
{% highlight java %}
List<Group> groups = user1.getGroups();
{% endhighlight %}
Then, the output for *groups* will be:
{% highlight java %}
[Group [name=group1], Group [name=group2]]    
{% endhighlight %}
But, what can we do easily when we need to get the groups of a user whose group role is only *MODERATOR?*

In the next section, we will figure out an easy way of doing this.

## 6. Custom Filters on Relation Entity
Since we have already defined *UserGroupRelation* as an entity, we always have the chance to fetch groups by writing custom **JPQL queries**.

However, we have a quicker way of doing this. **We can use JPA's *@WhereJoinTable* annotation and define another property which directly fetches only the filtered groups.**

Let's define *moderatorGroups* property and put *@WhereJoinTable* annotation configured with our expected role filter:
{% highlight java %}
@WhereJoinTable(clause = "role='MODERATOR'")
@ManyToMany
@JoinTable(
	name = "r_user_group",
	joinColumns = @JoinColumn(name = "user_id"),
	inverseJoinColumns = @JoinColumn(name = "group_id")
)
private List<Group> moderatorGroups = new ArrayList<>();
{% endhighlight %}
Thus, we can easily get the groups on which the user has only the role of *MODERATOR*.
{% highlight java %}
List<Group> groups = user1.getModeratorGroups();
{% endhighlight %}
So, this time the output will be:
{% highlight java %}
[Group [name=group1]]
{% endhighlight %}

## 7. Conclusion
In this tutorial, **we learned how to filter many-to-many collections by the columns of relation table with JPA *@WhereJoinTable* annotation**.

As always, all the code samples given in this tutorial are available [over on GitHub](https://github.com).
