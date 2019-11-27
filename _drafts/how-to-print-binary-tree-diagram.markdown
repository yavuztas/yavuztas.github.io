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
Despite being traditional, printing is the most common visualization technique for data structures. As well as being super common, printing can be tricky sometimes when it comes to tree structures that are specialized to represent hierarchies and structural relationships.

However, [Binary Tree](https://www.baeldung.com/java-binary-tree) is a special type of tree that has only two child nodes most and it is going to be the tree structure that we use in this article.

Therefore, we are going to learn how to print a useful diagram for Binary Trees in Java.

## 2. Different Tree Diagrams
Despite the limitations of drawing with only characters over on console, there can still be so many different diagram shapes to represent tree structures. Choosing one of them mostly depends on the size and the balance of the tree.

Let's see some of the possible types of diagrams which we can print:
{% highlight java %}
       1                     1              │       
      / \              ┌─────┴─────┐        │   ┌── 3                    
     /   \             2           3        │   │   
    2     3        ┌───┴───┐                └── 1
   / \             4       5                    │   ┌── 5        
  4   5                                         └── 2
                                                    └── 4
{% endhighlight %}
But, we will explain a practical one which is also easier to implement:
{% highlight java %}
root
├── node1
│   ├── node3
│   │   └── node5
│   └── node4
└── node2
{% endhighlight %}
We should notice that there are some neat benefits to choose a horizontal diagram over others.

**First, horizontal tree flows always to the same direction, left to right and top to bottom, just as the same as the text flows.** This automatically balances the tree view.

Consequently, we can show large and unbalanced trees as well.

Plus, we can always find the root in the same position which is the left upper corner.

**Second, the length of node values do not affect the visual structure.**

**Finally, it is much easier to implement.**

Therefore, we will go with a horizontal diagram and implement a simple binary tree printer class in the next sections.

## 3. Sample Binary Tree Model
First of all, we should model a basic binary tree which is super easy in Java.

Let's define a simple *BinaryTreeModel* class:
{% highlight java %}
public class BinaryTreeModel {

    private Object value;
    private BinaryTreeModel left;
    private BinaryTreeModel right;

    public BinaryTreeModel(Object value) {
        this.value = value;
    }

    // standard getters and setters

}
{% endhighlight %}

## 4. Creating Test Data
Before we start implementing our binary tree printer, we need to create a sample data to incrementally test it as visual:
{% highlight java %}
BinaryTreeModel root = new BinaryTreeModel("simple");

BinaryTreeModel node1 = new BinaryTreeModel("node1");
BinaryTreeModel node2 = new BinaryTreeModel("node2");
root.setLeft(node1);
root.setRight(node2);

BinaryTreeModel node3 = new BinaryTreeModel("node3");
BinaryTreeModel node4 = new BinaryTreeModel("node4");
node1.setLeft(node3);
node1.setRight(node4);

BinaryTreeModel node5 = new BinaryTreeModel("node5");
BinaryTreeModel node6 = new BinaryTreeModel("node6");
node2.setLeft(node5);
node2.setRight(node6);

BinaryTreeModel node7 = new BinaryTreeModel("node7");
node3.setLeft(node7);

BinaryTreeModel node8 = new BinaryTreeModel("node8");
BinaryTreeModel node9 = new BinaryTreeModel("node9");
node7.setLeft(node8);
node7.setRight(node9);
{% endhighlight %}

## 5. Implementing *BinaryTreePrinter*
Certainly, we need a separate helper class to keep our *BinaryTreeModel* clean for the sake of simplicity.

So, we define *BinaryTreePrinter* as our helper class:
{% highlight java %}
public class BinaryTreePrinter {

    private BinaryTreeModel tree;

    public BinaryTreePrinter(BinaryTreeModel tree) {
        this.tree = tree;
    }

}
{% endhighlight %}

### 5.1. Pre-Order Traversal
Considering our horizontal diagram, to print it properly, we can make a simple start by using **pre-order** traversal.

Consequently, **to perform pre-order traversal, we need to implement a recursive method that visits the root node at first, then left and finally right subtrees in order.**

Let's define a method to traverse our tree:
{% highlight java %}
public void traversePreOrder(StringBuilder sb, BinaryTreeModel node) {
    if (node != null) {

        sb.append(node.getValue());
        sb.append("\n");

        traversePreOrder(sb, node.getLeft());
        traversePreOrder(sb, node.getRight());

    }
}
{% endhighlight %}
Next, define another method to print:
{% highlight java %}
public void print() {
    StringBuilder sb = new StringBuilder();
    traversePreOrder(sb, this.tree);
    System.out.println(sb.toString());
}
{% endhighlight %}
Thus, we can simply print our test tree:
{% highlight java %}
new BinaryTreePrinter(root).print();
{% endhighlight %}
The output will be the list of tree nodes in traversed order:
{% highlight java %}
root
node1
node3
node7
node8
node9
node4
node2
node5
node6
{% endhighlight %}

### 5.2. Adding Tree Edges
To shape up our diagram nicely, we use three types of characters *├──*, *└──*, *│* to visualize nodes, the first two of them are for pointers and the last one is to fill the edges and connect the pointers.

Let's update our *traversePreOrder* method, add a *padding* parameter and use *│* character as padding value:
{% highlight java %}
public void traversePreOrder(StringBuilder sb, String padding, BinaryTreeModel node) {
    if (node != null) {

        sb.append(padding);
        sb.append(node.getValue());
        sb.append("\n");

        StringBuilder paddingBuilder = new StringBuilder(padding);
        paddingBuilder.append("│  ");

        traversePreOrder(sb, paddingBuilder.toString(), node.getLeft());
        traversePreOrder(sb, paddingBuilder.toString(), node.getRight());

    }
}
{% endhighlight %}
Also, we update *print* method as well:
{% highlight java %}
public void print() {
    StringBuilder sb = new StringBuilder();
    traversePreOrder(sb, "", this.tree);
    System.out.println(sb.toString());
}
{% endhighlight %}

So, let's test our *BinaryTreePrinter* again:
{% highlight java %}
root
│  node1
│  │  node3
│  │  │  node7
│  │  │  │  node8
│  │  │  │  node9
│  │  node4
│  node2
│  │  node5
│  │  node6
{% endhighlight %}
As we can see, we correctly implemented paddings but also need to continue with pointers.

Hence, we add another parameter *pointer* for our method *traversePreOrder*:
{% highlight java %}
public void traversePreOrder(StringBuilder sb, String padding, String pointer, BinaryTreeModel node) {
    if (node != null) {

        sb.append(padding);
        sb.append(pointer);
        sb.append(node.getValue());
        sb.append("\n");

        StringBuilder paddingBuilder = new StringBuilder(padding);
        paddingBuilder.append("│  ");

        String paddingForBoth = paddingBuilder.toString();
        String pointerForBoth = "├──";

        traversePreOrder(sb, paddingForBoth, pointerForBoth, node.getLeft());
        traversePreOrder(sb, paddingForBoth, pointerForBoth, node.getRight());

    }
}
{% endhighlight %}
Since we don't need any pointer for root, we pass only emtpy string in *print* method:
{% highlight java %}
public void print() {
    StringBuilder sb = new StringBuilder();
    traversePreOrder(sb, "", "", this.tree);
    System.out.println(sb.toString());
}
{% endhighlight %}

Let's see the output again:
{% highlight java %}
root
│  ├──node1
│  │  ├──node3
│  │  │  ├──node7
│  │  │  │  ├──node8
│  │  │  │  ├──node9
│  │  ├──node4
│  ├──node2
│  │  ├──node5
│  │  ├──node6
{% endhighlight %}
Some nodes don't have both left and right edges but only left like *node3* in our output. Besides, right edges cannot have another sibling because they have only left like *node2*.

Therefore, we should use another character *└──* for the right edges as pointers.

Let's make a small fix in *traversePreOrder* method:
{% highlight java %}
public void traversePreOrder(StringBuilder sb, String padding, String pointer, BinaryTreeModel node) {
    if (node != null) {

        sb.append(padding);
        sb.append(pointer);
        sb.append(node.getValue());
        sb.append("\n");

        StringBuilder paddingBuilder = new StringBuilder(padding);
        paddingBuilder.append("│  ");

        String paddingForBoth = paddingBuilder.toString();
        String pointerForRight = "└──";
        String pointerForLeft = (node.getRight() != null) ? "├──" : "└──";

        traversePreOrder(sb, paddingForBoth, pointerForLeft, node.getLeft());
        traversePreOrder(sb, paddingForBoth, pointerForRight, node.getRight());

    }
}
{% endhighlight %}
Now the output is correct for pointers:
{% highlight java %}
root
│  ├──node1
│  │  ├──node3
│  │  │  └──node7
│  │  │  │  ├──node8
│  │  │  │  └──node9
│  │  └──node4
│  └──node2
│  │  ├──node5
│  │  └──node6
{% endhighlight %}
However, we still have some extra lines to get rid of:
{% highlight java %}
  root
1.│  ├──node1
  │  │  ├──node3
  │  │  │  └──node7
  │  │  │3.│  ├──node8
  │  │  │  │  └──node9
  │  │  └──node4
  │  └──node2
  │2.│  ├──node5
  │  │  └──node6
{% endhighlight %}
As we mark over on diagram, there are still extra lines:
1. for root
2. precedes the nodes under a right subtree
3. precedes the single left nodes without right under a left subtree

### 5.3. Different Implementations For Root and Child Nodes
In order to fix extra lines under the root node, we can split our traverse method as one for root and other for child nodes to apply different behaviors.

Let's customize *traversePreOrder* for only root node and create another method as *traverseNodes* to traverse child nodes:
{% highlight java %}
public String traversePreOrder(BinaryTreeModel root) {

    if (root == null) {
        return "";
    }

    StringBuilder sb = new StringBuilder();
    sb.append(root.getValue());
    sb.append("\n");

    String pointerRight = "└──";
    String pointerLeft = (root.getRight() != null) ? "├──" : "└──";

    traverseNodes(sb, "", pointerLeft, root.getLeft(), root.getRight() != null);
    traverseNodes(sb, "", pointerRight, root.getRight(), false);

    return sb.toString();
}

public void traverseNodes(StringBuilder sb, String padding, String pointer, BinaryTreeModel node) {
    if (node != null) {

        sb.append(padding);
        sb.append(pointer);
        sb.append(node.getValue());
        sb.append("\n");

        StringBuilder paddingBuilder = new StringBuilder(padding);
        paddingBuilder.append("│  ");

        String paddingForBoth = paddingBuilder.toString();
        String pointerForRight = "└──";
        String pointerForLeft = (node.getRight() != null) ? "├──" : "└──";

        traverseNodes(sb, paddingForBoth, pointerForLeft, node.getLeft());
        traverseNodes(sb, paddingForBoth, pointerForRight, node.getRight());

    }
}
{% endhighlight %}
Also, we need a small change in our *print* method:
{% highlight java %}
public void print() {
    System.out.println(traversePreOrder(tree));
}
{% endhighlight %}

Thus, the lines under the root node disappeared and the output is better now:
{% highlight java %}
root
├──node1
│  ├──node3
│  │  └──node7
│  │  │  ├──node8
│  │  │  └──node9
│  └──node4
└──node2
│  ├──node5
│  └──node6
{% endhighlight %}
But we have still extra lines left for child nodes. As we notice, if only a left node has any sibling then we should use *│* character. Otherwise, we left blank to remove the extra lines.

So, let's add a new parameter *hasRightSibling* to *traverseNodes* and implement the required behavior:
{% highlight java %}
public void traverseNodes(StringBuilder sb, String padding, String pointer, BinaryTreeModel node, boolean hasRightSibling) {

    if (node != null) {

        sb.append(padding);
        sb.append(pointer);
        sb.append(node.getValue());
        sb.append("\n");

        StringBuilder paddingBuilder = new StringBuilder(padding);
        if (hasRightSibling) {
            paddingBuilder.append("│  ");
        } else {
            paddingBuilder.append("   ");
        }

        String paddingForBoth = paddingBuilder.toString();
        String pointerRight = "└──";
        String pointerLeft = (node.getRight() != null) ? "├──" : "└──";

        traverseNodes(sb, paddingForBoth, pointerLeft, node.getLeft(), node.getRight() != null);
        traverseNodes(sb, paddingForBoth, pointerRight, node.getRight(), false);

    }

}
{% endhighlight %}
Finally, our diagram reached the expected shape that we need and we acquire a nice and clean output:
{% highlight java %}
root
├──node1
│  ├──node3
│  │  └──node7
│  │     ├──node8
│  │     └──node9
│  └──node4
└──node2
   ├──node5
   └──node6
{% endhighlight %}

## 6. Conclusion
In this article, **we learned a simple and practical way to print Binary Tree Diagram in Java**.

Like always, all the source code for examples are available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java).
