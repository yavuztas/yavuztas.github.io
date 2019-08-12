---
layout: post
title:  "Consuming Variable Responses in Jackson"
date:   2014-11-30 14:34:25
categories: java jackson
tags: regular
image: /assets/article_images/consuming-variable-responses-in-jackson/header-java.png
image2: /assets/article_images/consuming-variable-responses-in-jackson/header-java-mobile.png
---
### 1. Overview
This post is a follow up for our previous [post]() which was about handling variable responses in **Gson**.
Today, we will implement the same solution by using another popular library of JSON in Java, [**the Jackson library**](https://github.com/FasterXML/jackson). To get information about the usage of **Jackson** in detail, you may consider reading [Jackson JSON Tutorial](https://www.baeldung.com/jackson).

Before we start to use the library, we need to add the Maven dependencies into our project's pom.xml:
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
Note that if we are already using **Spring Boot** and **Spring Web Starter** module enabled in our project then this means we already have the necessary dependencies and do not need to add any extra dependency for **Jackson**.
### 2. Sample Response
Let's use the same sample API responses from the previous post:
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
And again, if there is only one single comment, response structure changes:
{% highlight json %}
{
    "id": 1,
    "name": "sample article",
    "comments": {"id":1, "text":"some comment text"}
}
{% endhighlight %}
### 3. Writing a Response Model
We will use the same model for *article*:
{% highlight java %}
public class ArticleModel {

    private Long id;

    private String name;

    private List<CommentModel> comments;

    // standard getters and setters

}
{% endhighlight %}
And for *comment* as well:
{% highlight java %}
public class CommentModel {

    private Long id;

    private String text;

    // standard getters and setters

}
{% endhighlight %}
### 4. Consuming the response
We use a type of collection for the *comment* field as *List&lt;CommentModel&gt;* thus we can handle automatically for multiple comments which *Jackson* already does the heavy lifting as the same as its competitor *Gson*:
{% highlight java %}
String jsonString = "{"id": 1,"name": "sample article","comments": [{"id":1, "text":"some comment text"},{"id":2, "text":"some another comment text"}]}";

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
Again, although this configuration handles for the multiple values of comments if the response changes to a single value then it simply does not work.
Consequently, we need to define a custom *JsonDeserializer* to handle both single and multiple values.

### 5. Custom *JsonDeserializer* for Jackson
Because we aim to implement a custom deserialization behavior, we need to create a particular *JsonDeserializer*. This behavior will be responsible for adding any single value to the list as the same as multiple values are being added automatically by *Jackson*.

So Let's create a *JsonDeserializer* for this purpose:
{% highlight java %}
public class CommentListDeserializer extends StdDeserializer<List> {

	public CommentListDeserializer() {
		this(null);
	}

	private CommentListDeserializer(JavaType valueType) {
		super(valueType);
	}

	@Override
	public List deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

		return null;
	}

}
{% endhighlight %}
We need a *TypeParameter* for Jackson to specify the inner type and the type of *Collection* at once.

First, let's define a constant for a type of *List&lt;CommentModel&gt;*:
{% highlight java %}

private static final TypeReference<List<CommentModel>> LIST_OF_COMMENTS_TYPE = new TypeReference<List<CommentModel>>() {};

{% endhighlight %}
Then, we implement the *deserialize* method of our deserializer to culminate in our expected behavior:
{% highlight java %}
@Override
public List deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {

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
### 6. Using Deserialization Features in Jackson
There are lots of features for conversion which are bundled in Jackson library. These predefined features provide efficient ways to customize both serialization and deserialization without writing code. This time we mostly focus on deserialization features and look forward to finding anything to provide the same behavior by the power of Jackson's features.

We can simply enable or disable any of these features as globally:
{% highlight java %}

ObjectMapper mapper = new ObjectMapper();
mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
mapper.enable(DeserializationFeature.ACCEPT_SINGLE_VALUE_AS_ARRAY);

{% endhighlight %}
Besides some of these features can be set on class or property level by the help of the *@JsonFormat* annotation. In our example, we can use the *ACCEPT_SINGLE_VALUE_AS_ARRAY* feature to provide the same behavior automatically.

Let's remove our custom deserializer and instead add a simple *@JsonFormat* annotation to our *comments* field in *ArticleModel*:
{% highlight java %}

@JsonFormat(with = Feature.ACCEPT_SINGLE_VALUE_AS_ARRAY)
private List<CommentModel> comments;

{% endhighlight %}
And that's it. This will automatically add any single value of *CommentModel* to our list as if it were already inside an array. If we do not need any further customization this is the most appropriate solution for our specific case.

For the full list of features that can be used with *@JsonFormat* annotation, you can check the [source code](https://fasterxml.github.io/jackson-annotations/javadoc/2.9/com/fasterxml/jackson/annotation/JsonFormat.Feature.html).

### 7. Creating a Generic Deserializer
We may want to implement something different that we cannot handle with predefined features. Also for the benefit of reusability, we may want to proceed further by refactoring our code to prevent creating a new deserializer for each content type. On some occasions, it might be essential to go on with this method.

First, we will implement another *JsonDeserializer* to handle the deserialization in a generic and more customizable way:
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
		// if token is array type then perform object deserialization to each content element
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
		// just use jackson default object deserializer by using content type
		return p.readValueAs(contentClassType);
	}

}
{% endhighlight %}
Hence we can use the same deserializer for different content types when we need it.

Again the problem we encounter here is to obtain the class type of the content inside our collection. Without passing this info it is not possible for Jackson to initialize the correct concrete type but only *LinkedHashMap*. However, there is a workaround Jackson provides us to obtain bean related information inside our deserializer. We implemented the *ContextualDeserializer* interface to inform Jackson that we need information about the bean we are dealing with. Thus an instance of *BeanProperty* is automatically injected and we obtained the class type of the content.

In the end, to use our generic deserializer, let's change our *comments* field:
{% highlight java %}

@JsonDeserialize(using = SingleAwareListDeserializer.class)
private List<CommentModel> comments;

{% endhighlight %}
In conclusion, with the abilities of Jackson library, we can provide the same behavior, *SingleAwareListDeserializer*, for the other properties as well. As a result, this will lead us to our purpose and provide an effective way of **reusability**.

### 8. Conclusion
In this tutorial, **we learned how to implement custom JSON deserializers to handle variable responses in Jackson**.

As always, the examples shown in this tutorial are available [over on GitHub](https://github.com).
