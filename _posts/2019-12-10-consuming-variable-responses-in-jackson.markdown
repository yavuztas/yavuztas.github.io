---
layout: post
title:  "Consuming Variable Responses in Jackson"
date:   2019-12-10 22:00:00
readtime: 6 min read
categories: java jackson
tags: featured
image: /assets/article_images/2019-12-10-consuming-variable-responses-in-jackson/variable-responses-jackson.jpg
image2: /assets/article_images/2019-12-10-consuming-variable-responses-in-jackson/variable-responses-jackson-mobile.jpg
excerpt: In addition to being another popular library of JSON in Java, the <b>Jackson Library</b> is very well known for its ability to offer deep customization in an opinionated way.
citation: Photo by <a href="https://unsplash.com/@franckinjapan?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Franck V.</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
---
In addition to being another popular library of <i>JSON</i> in Java, **the [Jackson Library](https://github.com/FasterXML/jackson) is very well known for its ability to offer deep customization in an opinionated way.**

In this article, we are going to show an alternate way by using the *Jackson Library* for the same solution which was about [how to handle variable responses in Gson](https://yavuztas.dev/java/gson/2019/10/12/consuming-variable-responses-in-gson.html).

## Adding Dependencies
Before we start, we need to add the Maven dependencies into our project's *pom.xml:*
{% highlight xml %}

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.9</version>
</dependency>

<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.9</version>
</dependency>

{% endhighlight %}
We should note that if we already use **Spring Boot** and **Spring Web Starter** module enabled in our project then simply, we don't need to add any extra dependency for Jackson.
## Sample Response
Let's use the sample API response:
{% highlight json %}
{
    "id": 1,
    "name": "sample article",
    "comments": [
        {"id":1, "text":"some comment text"},
        {"id":2, "text":"some another comment text"}
    ]
}
{% endhighlight %}
Also, the response changes from *Array* to *Object* notation when there is only a single *comment:*
{% highlight json %}
{
    "id": 1,
    "name": "sample article",
    "comments": {"id":1, "text":"some comment text"}
}
{% endhighlight %}
## Response Models
We'll also use the same models from the [previous article](https://yavuztas.dev/java/gson/2019/10/12/consuming-variable-responses-in-gson.html):
{% highlight java %}
public class ArticleModel {

    private Long id;

    private String name;

    private List<CommentModel> comments;

    // standard getters and setters

}

public class CommentModel {

    private Long id;

    private String text;

    // standard getters and setters

}
{% endhighlight %}
## Consuming the Response
We defined a collection type for the *comment* field as *List&lt;CommentModel&gt;*. Thus, we can map multiple comments into a single field automatically because *Jackson* already does the heavy lifting for us:
{% highlight java %}
String jsonString = "{
    "id": 1,
    "name": "sample article",
    "comments": [
        {"id":1, "text":"some comment text"},
        {"id":2, "text":"some another comment text"}
    ]
}";

ObjectMapper mapper = new ObjectMapper();
ArticleModel model = mapper.readValue(this.mockResponseMultiValue, ArticleModel.class);

Assert.assertEquals(ArticleModel.class, model.getClass());
Assert.assertEquals(model.getComments().getClass(), CommentList.class);
Assert.assertEquals(model.getComments().get(0).getClass(), CommentModel.class);

Assert.assertEquals(1, model.getId().longValue());
Assert.assertEquals("sample article", model.getName());

Assert.assertEquals(2, model.getComments().size());
Assert.assertEquals(1, model.getComments().get(0).getId().longValue());
Assert.assertEquals("some comment text", model.getComments().get(0).getText());
Assert.assertEquals(2, model.getComments().get(1).getId().longValue());
Assert.assertEquals("some another comment text", model.getComments().get(1).getText());
{% endhighlight %}
Certainly, this configuration resolves the multiple comment values unless the response changes to a single value structure. Otherwise, it simply doesn't work.

Consequently, we need to define a custom deserializer to handle both single and multiple values at once.

## Custom Deserializers for Jackson
*StdDeserializer* is an abstract type and also the base class for common deserializers in Jackson.

Therefore, to implement a custom deserialization behavior, we will create an implementation of *StdDeserializer*. This behavior will be responsible for adding any single value to the list as the same as multiple values are being added automatically by Jackson.

So, let's create *CommentListDeserializer* which inherits the base class *StdDeserializer*:
{% highlight java %}
public class CommentListDeserializer extends StdDeserializer<List> {

    public CommentListDeserializer() {
        this(null);
    }

    private CommentListDeserializer(JavaType valueType) {
        super(valueType);
    }

    @Override
    public List deserialize(JsonParser p, DeserializationContext ctxt)
      throws IOException, JsonProcessingException {

        return null;
    }

}
{% endhighlight %}
We need a *TypeParameter* for Jackson to specify the *Collection* type and the type of elements in which are involved as well.

So, let's define a constant for a type of *List&lt;CommentModel&gt;*:
{% highlight java %}

private static final TypeReference<List<CommentModel>> LIST_OF_COMMENTS_TYPE =
  new TypeReference<List<CommentModel>>() {};

{% endhighlight %}
Next, we implement the *deserialize* method to come up with the expected behavior:
{% highlight java %}
@Override
public List deserialize(JsonParser p, DeserializationContext ctxt)
  throws IOException, JsonProcessingException {

    List<CommentModel> commentList = new ArrayList<>();

    JsonToken token = p.currentToken();

    if (JsonToken.START_ARRAY.equals(token)) {
        List<CommentModel> listOfArticles = p.readValueAs(LIST_OF_COMMENTS_TYPE);
        commentList.addAll(listOfArticles);
    }

    if (JsonToken.START_OBJECT.equals(token)) {
        CommentModel article = p.readValueAs(CommentModel.class);
        commentList.add(article);
    }

    return commentList;
}
{% endhighlight %}
## Deserialization Features in Jackson
Although writing a custom deserializer offers us great flexibility, there are lots of features bundled in the Jackson Library for conversions. **Jackson predefined features provide practical ways to customize both serialization and deserialization.**

Hopefully, *ACCEPT_SINGLE_VALUE_AS_ARRAY* feature does the exact job for our use-case. Certainly, we could use it for the sake of simplicity:
{% highlight java %}

ObjectMapper mapper = new ObjectMapper();
mapper.enable(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY);

{% endhighlight %}
Also, **an alternate way is to configure the *@JsonFormat* annotation.** In our example, we'll go with the *@JsonFormat* annotation to set the *ACCEPT_SINGLE_VALUE_AS_ARRAY* feature.

Let's remove our custom deserializer and add a simple *@JsonFormat* annotation to our *comments* field in *ArticleModel*:
{% highlight java %}

@JsonFormat(with = Feature.ACCEPT_SINGLE_VALUE_AS_ARRAY)
private List<CommentModel> comments;

{% endhighlight %}
Thus, this will automatically add any single value of *CommentModel* to our list as if it were already inside an array. **Unless we have any further requirement using Jackson features can be the most elegant solution for our use-case.**

We can always have a chance to check the full list of features over on the [source code](https://fasterxml.github.io/jackson-annotations/javadoc/2.9/com/fasterxml/jackson/annotation/JsonFormat.Feature.html).

## Going Further with Generic Deserializers
We can write generic deserializers for different requirements that we couldn't handle by predefined features. Besides, for the benefit of reusability, we can go further to refactor our code not to create new deserializers for each element type.

So, we'll implement a new class as *SingleAwareListDeserializer* to handle the deserialization in a more customizable way:
{% highlight java %}
public class SingleAwareListDeserializer extends StdDeserializer<List> implements ContextualDeserializer {

    private Class<?> contentClassType;

    public SingleAwareListDeserializer() {
        this(null);
    }

    private SingleAwareListDeserializer(JavaType valueType) {
        super(valueType);
    }

    @Override
    public JsonDeserializer<?> createContextual(DeserializationContext ctxt, BeanProperty property) throws JsonMappingException {
        // we use ContextualDeserializer to obtain content class type
        contentClassType = property.getType().getContentType().getRawClass();
        return this;
    }

    @Override
    public List deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

        List list = new ArrayList<>();

        JsonToken token = p.currentToken();
        // if token is array type then perform object deserialization to each element element
        if (JsonToken.START_ARRAY.equals(token)) {
            while (p.nextToken() != null) {
                if (JsonToken.START_OBJECT.equals(p.currentToken())) {
                    list.add(deserializeObject(p));
                }
            }
        }

        // if token is object type
        if (JsonToken.START_OBJECT.equals(token)) {
            list.add(deserializeObject(p));
        }

        return list;
    }

    private Object deserializeObject(JsonParser p) throws IOException {
        // just use jackson default object deserializer by using element type
        return p.readValueAs(contentClassType);
    }

}
{% endhighlight %}
Now, we can use the same deserializer for different types as well.

Certainly, we should notice the challenge here which is about how to obtain the class type of the element inside our collection. **Without the element's type information, it is not possible for Jackson to initialize the concrete type but only *LinkedHashMap*.**

However, there is a workaround to obtain type-related information inside our deserializer.

**We used the *ContextualDeserializer* interface to inform Jackson that we need information about the bean we are dealing with.** Thus, an instance of *BeanProperty* is automatically injected and we obtained the class type of the collection element.

Finally, to use our generic deserializer, let's change our *comments* field:
{% highlight java %}

@JsonDeserialize(using = SingleAwareListDeserializer.class)
private List<CommentModel> comments;

{% endhighlight %}
As a result, we can provide the same behavior by the *SingleAwareListDeserializer* generically for all types as well.

## Finally
In this tutorial, **we learned how to handle variable responses in Jackson by using the builtin features and custom deserializers as well**.

All the source code for the examples shown in this tutorial are available [over on GitHub](https://github.com/yavuztas/java-jackson-multivalue).
