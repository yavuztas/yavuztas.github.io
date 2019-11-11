---
layout: post
title:  "How to Print Binary Tree Diagram"
date:   2020-11-30 14:34:25
categories: java
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## 1. Introduction
One of the most effective data structures are **Trees** when it comes to represent structural relationships and hierarchicies.
//TODO Transition to Binary Trees...
In this tutorial, we are going to learn how to print a useful diagram for Binary Trees in Java.

## 2. What is a Binary Tree?
Binary Tree is a special type of Trees which is formed by two edges, left and right, and also the data itself.

Let's show a sample diagram for a simple binary tree:
{% highlight java %}
      1
     / \
    2   3
{% endhighlight %}

## 3. Different Tree Diagrams
Despite the limitations of drawing with only characters over on console, there can still be so many different diagram shapes to represent tree structures. It mostly dependes on the balance and the size of the tree.

Let's see some of the different types of console prints for binary trees:
{% highlight java %}
       1
      / \
     /   \
    2     3
   / \   / \
  4   5 6   7 ... put the other ones
{% endhighlight %}

However, we will explain the most practical one which is also easier to implement:
https://stackoverflow.com/a/43348945/7794994
{% highlight java %}
node1
├── node2
│   ├── node4
│   │   └── node6
│   └── node5
└── node3
{% endhighlight %}

We choose a horizontal diagram because of some benefits:
 * Tree flows allways to the same direction, left to right, top to bottom, just as the same as text flow. This automatically balances the tree.
 * The length of node values do not affect the visual structure.
 * Can show large and unbalanced Trees as well. We can always find the root at the left upper corner.
 * Much easier to implement

## 4. Sample Binary Tree Model
 Modeling a basic binary tree is very trival in Java.

 Let's define a simple *BinaryTree* class with its structures:
 {% highlight java %}
 public class BinaryTree {

   private Object value;
   private BinaryTree left;
   private BinaryTree right;

 }
 {% endhighlight %}

## 5. Creating Test Data

## 6. Implementing *BinaryTreePrinter*

## 7. Running Tests

## 8. Conclusion
In this tutorial, **we learned a simple way to print Binary Tree Diagram in Java**.

All the sample codes showed in this tutorial are available [over on GitHub](https://github.com).
