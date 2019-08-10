---
layout: post
title:  "Writing a Utility Class for Collections in Java 8"
date:   2019-08-10 18:00:00
categories: java collections streams
tags: regular
image: /assets/article_images/2019-08-10-java8-utility-class/java8-utility-class.jpg
image2: /assets/article_images/2019-08-10-java8-utility-class/java8-utility-class.jpg
excerpt: One of the fluent APIs that Java 8 brings us, is the <b>Java 8 Stream API</b>, which provides a practical and minimal interface to code, especially with the help of <b>Java 8 Lambdas</b>.
---
One of the fluent APIs that Java 8 brings us, is the [Java 8 Stream API](https://www.baeldung.com/java-8-streams), which provides a practical and minimal interface to code, especially with the help of [Java 8 Lambdas](https://www.baeldung.com/java-8-lambda-expressions-tips).

In this tutorial, we are going to write a simple utility class to convert and modify java collections with ease, by using the power of **Java 8 Streams**.

### Utility Classes
Utility classes are structures that contain reusable and stateless helper methods. We implement these methods for handling any kind of specific operations for specific purposes. Because of their stateless nature, we mostly prefer to define them as *static*.

First of all, let's define a simple class:
{% highlight java %}
public class UtilsForCollections {

}
{% endhighlight %}
### Static Classes in Java
Static classes are classes that cannot be instantiated. As utility classes do not need to hold any state, it is appropriate to define them as static to prevent creating instances of them.

Unlike some other languages, in Java, we cannot use the *static* modifier for classes. Instead, we will use a **private constructor** to give the same ability to our utility classes:
{% highlight java %}
public class UtilsForCollections {

    private UtilsForCollections() {

    }

}
{% endhighlight %}
### Array to Collection Conversion
The first method of our utility class will be a *generic* method that simply converts a given *array* structure to *java.util.Collection* type.

We have two different behavioral types to return as java collections, one of them is *List* that is used when we need to preserve the **order** of the elements, the other one is *Set* that is used when we need to preserve the **uniqueness** of the elements.

First, let's define a generic method that handles *array* to *List* conversion:
{% highlight java %}

public static <T> List<T> toList(T[] array) {
    // TODO implement but how?
}

{% endhighlight %}
### Power of Java 8 Stream API
Normally to implement this kind of method, we need to traverse the elements of an array and populate each of them to a newly created *ArrayList* object. However, thanks to the **Java 8 Stream API**, we can easily get rid of this sort of boilerplates.

So let's implement our *toList* method:
{% highlight java %}

public static <T> List<T> toList(T[] array) {
    return Arrays.stream(array).collect(Collectors.toList());
}

{% endhighlight %}
We use the *Arrays.stream()* method to open a stream over the given array. Once we acquire the stream instance, we may operate over the stream by many provided methods like *collect*, *filter*, *map*, etc. We choose the method *collect* here to copy elements from an array to another structure with the help of predefined **stream collectors**.

Like *toList* method, let's define a separate method for *array* to *Set* conversion:
{% highlight java %}

public static <T> Set<T> toSet(T[] array) {
    return Arrays.stream(array).collect(Collectors.toSet());
}

{% endhighlight %}
### Handling Primitive Types
With the help of *generics* we successfully handled for all object types but when it comes to primitive types of java we cannot use our generic methods. We see an error when we try to use with primitive-typed arrays and our java compiler complains about the argument types which are not applicable:
{% highlight java %}

int[] intArray = new int[] { 1, 2, 3 };
UtilsForCollections.toList(intArray); // this line gives us a compile error!

{% endhighlight %}
One of the ways of solving this problem is simply defining separate methods with primitive-typed arguments. We will use *boxed* method of *streams* to box every primitive element to its wrapper objects:
{% highlight java %}

public static List<Integer> toList(int[] array) {
    return Arrays.stream(array).boxed().collect(Collectors.toList());
}

public static List<Long> toList(long[] array) {
    return Arrays.stream(array).boxed().collect(Collectors.toList());
}

public static List<Double> toList(double[] array) {
    return Arrays.stream(array).boxed().collect(Collectors.toList());
}

{% endhighlight %}
We can define more methods in the same way for other remaining primitives (*boolean*, *byte*, *char*, *short*, *float*) if we need.

And also we can define their *Set* versions that are pretty same:
{% highlight java %}

public static Set<Integer> toSet(int[] array) {
    return Arrays.stream(array).boxed().collect(Collectors.toSet());
}

public static Set<Long> toSet(long[] array) {
    return Arrays.stream(array).boxed().collect(Collectors.toSet());
}

public static Set<Double> toSet(double[] array) {
    return Arrays.stream(array).boxed().collect(Collectors.toSet());
}

{% endhighlight %}
### Ordered Sets
Sometimes it might be important to preserve both the order and the uniqueness of the elements and we need to handle this with a single collection type. If we encounter this problem in java the *LinkedHashSet* collection type comes to rescue.

Let's add an optional parameter to our *ToSet* method in order to return *LinkedHashSet* type instead of unordered *HashSet*:
{% highlight java %}

public static <T> Set<T> toSet(T[] array) {
    return UtilsForCollections.toSet(array, false);
}

public static <T> Set<T> toSet(T[] array, boolean preserveOrder) {
    if (preserveOrder) {
        return Arrays.stream(array)
            .collect(Collectors.toCollection(LinkedHashSet::new));
    }
    return Arrays.stream(array).collect(Collectors.toSet());
}

{% endhighlight %}
The former method here simply uses the overloaded method of itself. By doing this, we not only provide a default value for *preserveOrder* parameter but also protect the backward compatibility of our utility methods. We set *false* to *preserveOrder* as default value here.
### Collection to Collection Conversions
Conversion from a collection type to another collection type is also easily possible with streams. We can get streams from collections as well and we can perform *List* to *Set* conversion and vice versa.

Let's define some methods to add this ability to our utility class:
{% highlight java %}

public static <T> List<T> toList(Collection<T> collection) {
    return collection.stream().collect(Collectors.toList());
}

public static <T> Set<T> toSet(Collection<T> collection) {
    return UtilsForCollections.toSet(collection, false);
}

public static <T> Set<T> toSet(Collection<T> collection, boolean preserveOrder) {
    if (preserveOrder) {
        return collection.stream()
            .collect(Collectors.toCollection(LinkedHashSet::new));
    }
    return collection.stream().collect(Collectors.toSet());
}

{% endhighlight %}
### Filtering Arrays and Collections
One of the handy methods of Stream API is *filter* method. This provides a comfortable way of filtering an array or collection and creating subsets of theirs. With the power of **lambdas** we can also customize the behavior.

Let's define two methods to perform filtering on arrays or collections:
{% highlight java %}

public static <T> List<T> filterArray(T[] array, Predicate<? super T> predicate) {
    return Arrays.stream(array).filter(predicate)
        .collect(Collectors.toCollection(ArrayList::new));
}

public static <T> List<T> filterCollection(Collection<T> collection, Predicate<? super T> predicate) {
    return collection.stream().filter(predicate)
        .collect(Collectors.toCollection(ArrayList::new));
}

{% endhighlight %}
Next, let's use this method to filter an array of strings by their first letters as the character **" a "**:
{% highlight java %}

String[] array = new String[] { "apple", "banana", "orange", "avocado", "mango" };
List<String> filtered = UtilsForCollections.filterArray(array, v -> v.startsWith("a"));
filtered.forEach(System.out::println);

{% endhighlight %}
Now, by running this code, we will see the outputs of the filtered elements:
{% highlight java %}
[apple, avocado]
{% endhighlight %}

### Concatenate Elements into a String
Another ability that may be useful is knowing the elements of a collection or array structures. Besides we use it frequently for debugging purposes. We can extend our utility class by adding some methods to simply joining the string representations of the elements besides a custom separator and optional limit parameter.

So, let's define our join methods:
{% highlight java %}

public static <T> String join(T[] array, String separator) {
    return UtilsForCollections.join(array, separator, 0);
}

public static <T> String join(T[] array, String separator, int limit) {
    if (limit > 0 && array.length > limit) {
        return Arrays.stream(array).limit(limit).map(String::valueOf)
            .collect(Collectors.joining(separator)) + separator + "...";
    }
    return Arrays.stream(array).map(String::valueOf)
        .collect(Collectors.joining(separator));
}

public static <T> String join(Collection<T> collection, String separator) {
    return UtilsForCollections.join(collection, separator, 0);
}

public static <T> String join(Collection<T> collection, String separator, int limit) {
    if (limit > 0 && collection.size() > limit) {
        return collection.stream().limit(limit).map(String::valueOf)
            .collect(Collectors.joining(separator)) + separator + "...";
    }
    return collection.stream().map(String::valueOf)
        .collect(Collectors.joining(separator));
}

{% endhighlight %}

Then, let's use this *join* method with a limit of maximum **3** elements:
{% highlight java %}

String[] array = new String[] { "apple", "banana", "orange", "avocado", "mango" };
String joined = UtilsForCollections.join(array, ", ", 3);
System.out.println(joined);

{% endhighlight %}
Last, we will see the outputs of the joined elements:
{% highlight java %}
apple, banana, orange, ...
{% endhighlight %}
### Finally
Today, **we explained how to implement a simple utility class for collections by the help of Java 8 Stream API**.

There are many possible methods to implement which are likely to come in handy when we are dealing with collections.

So, what do you think about any other useful methods to include in our utility class? Just feel free to comment below!

All the code samples given in this tutorial are available [over on GitHub](https://github.com/yavuztas/java-utility-collections).
