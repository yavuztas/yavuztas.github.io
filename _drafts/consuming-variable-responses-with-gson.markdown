---
layout: post
title:  "Consuming Variable Responses in Gson"
date:   2014-11-30 14:34:25
categories: java gson
tags: regular
image: /assets/article_images/consuming-multi-valued-responses-with-gson/header-java.png
image2: /assets/article_images/consuming-multi-valued-responses-with-gson/header-java-mobile.png
---
### 1. Overview
No matter being public or private, Restful APIs are the most popular way to integrate our applications with the world outside. This means you do not have a chance to alter the services you consume, instead, you should adapt to them most of the time. Since Java is a static-typed language, it is sometimes a challenge while you are consuming and mapping the data into your response model object, especially if the response changes on some occasions.

Today, we will learn how to handle variable responses with [**the Gson library**](https://code.google.com/p/google-gson/). For the basic usage of **Gson** you may consider reading [gson serialization guide](https://www.baeldung.com/gson-serialization-guide) and [gson deserialization guide](https://www.baeldung.com/gson-deserialization-guide) before you start.

Before we start to use the library, we need to add the Maven dependency into our project's pom.xml:
{% highlight xml %}
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
{% endhighlight %}
### 2. Sample Response
Let's pick a sample API for a blog which exposes detailed information about its articles with the comments which may be more than one:
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
However, if there is only one single comment then response structure changes:
{% highlight json %}
{
    "id": 1,
    "name": "sample article",
    "comments": {"id":1, "text":"some comment text"}
}
{% endhighlight %}
### 3. Writing a Response Model
In order to map the data, we need a plain java object as a response model.
Let's start by defining a simple class that matches the main response structure:
{% highlight java %}
public class ArticleModel {

    @Expose
    private Long id;

    @Expose
    private String name;

    @Expose
    private List<CommentModel> comments;

    // standard getters and setters

}
{% endhighlight %}
And we need to define another class corresponds to *comment* response structure:
{% highlight java %}
public class CommentModel {

    @Expose
    private Long id;

    @Expose
    private String text;

    // standard getters and setters

}
{% endhighlight %}
### 4. Consuming the response
Since we use *List&lt;CommentModel&gt;* for the *comments* field, we can easily map multiple values for comments which Gson already does the heavy lifting for us:
{% highlight java %}
String jsonString = "{"id": 1,"name": "sample article","comments": [{"id":1, "text":"some comment text"},{"id":2, "text":"some another comment text"}]}";
Gson gson = new Gson();
ArticleModel model = gson.fromJson(jsonString, ArticleModel.class);

Assert.assertEquals(1, model.getId().longValue());
Assert.assertEquals("sample article", model.getName());
Assert.assertEquals(2, model.getComments().size());
Assert.assertEquals(1, model.getComments().get(0).getId().longValue());
Assert.assertEquals("some comment text", model.getComments().get(0).getText());
Assert.assertEquals(2, model.getComments().get(1).getId().longValue());
Assert.assertEquals("some another comment text", model.getComments().get(1).getText());
{% endhighlight %}
Although this configuration works for the multiple values of comments, if it changes to a single value then we need to define a custom *TypeAdapter* in order to handle both single and multiple values.
### 5. Custom *TypeAdapter* for Gson
Since our purpose is to implement a custom JSON deserialization behavior, we need to create a *TypeAdapter* to achieve this. This behavior will be responsible for adding any single value to the list as the same as multiple values are being added automatically by Gson.

So Let's create a Gson *TypeAdapter* for this purpose:
{% highlight java %}
public class CommentListTypeAdapter extends TypeAdapter<List> {

	@Override
	public void write(JsonWriter out, List list) throws IOException {

	}

	@Override
	public List read(JsonReader in) throws IOException {

		return null;
	}
}
{% endhighlight %}
We need to access *Gson* instance to preserve default deserialization behavior for *Object* and *Collection* types.

First, let's define a constructor and fields for default adapters:
{% highlight java %}

private Gson gson;
private TypeAdapter<CommentModel> objectTypeAdapter;
private TypeAdapter<List<CommentModel>> listTypeAdapter;

public CommentListTypeAdapter(Gson gson) {
    this.gson = gson;
    this.objectTypeAdapter = gson.getAdapter(CommentModel.class);
    this.listTypeAdapter = gson.getAdapter(new TypeToken<List<CommentModel>>() {});
}

{% endhighlight %}
Since we are aiming to do deserialization only, we can omit the *write* method of our adapter. However, we may implement in any case by simply delegating it to the default adapters.

Second, let's implement *write* method by using *listTypeAdapter*:
{% highlight java %}
@Override
public void write(JsonWriter out, List list) throws IOException {

    listTypeAdapter.write(out, list);

}
{% endhighlight %}
Last, we implement the *read* method of our adapter in order to come up with our expected deserialization behavior:
{% highlight java %}
@Override
public List read(JsonReader in) throws IOException {

	List<CommentModel> deserializedObject = new ArrayList<>();

	// type of next token
	JsonToken peek = in.peek();

	// if the json field is single object just add this object to list as an
	// element
	if (JsonToken.BEGIN_OBJECT.equals(peek)) {
		deserializedObject.add(deserializeObject(in));
	}

	// if the json field is array then implement normal array deserialization
	if (JsonToken.BEGIN_ARRAY.equals(peek)) {
		deserializedObject.addAll(deserializeList(in));
	}

	return deserializedObject;
}
{% endhighlight %}
### 6. *TypeAdapterFactory* in Gson
We may consider using *TypeAdapterFactory* in Gson. There are two benefits of this, one of them is we can implement a generic factory method to create different *TypeAdapter* classes for different types. The other is we can access the underlying *Gson* instance and reuse it which we are looking for primarily right now.  

So let's create a custom *TypeAdapterFactory*:
{% highlight java %}
public class CommentListTypeAdapterFactory implements TypeAdapterFactory {

	@Override
	public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {

		return (TypeAdapter<T>) new CommentListTypeAdapter(gson);
	}

}
{% endhighlight %}
We have just created an instance of our *CommentListTypeAdapter* and delegated the existing *Gson* instance to our type adapter.
### 7. Registering *TypeAdapterFactory*
Finally, we need to register our custom *TypeAdapterFactory* to the *Gson* context to make it work.

Let's update our *ArticleModel*:
{% highlight java %}

@JsonAdapter(CommentListTypeAdapterFactory.class)
@Expose
private List<CommentModel> comments;

{% endhighlight %}
Now we can consume single value structures like multiple values with the same *ArticleModel*:
{% highlight java %}

String jsonString = "{"id": 1,"name": "sample article","comments": {"id":1, "text":"some comment text"}}";
Gson gson = new Gson();
ArticleModel model = gson.fromJson(jsonString, ArticleModel.class);

Assert.assertEquals(1, model.getId().longValue());
Assert.assertEquals("sample article", model.getName());

Assert.assertEquals(1, model.getComments().size());
Assert.assertEquals(1, model.getComments().get(0).getId().longValue());
Assert.assertEquals("some comment text", model.getComments().get(0).getText());

{% endhighlight %}
### 8. Further Improvements With the Power of Generics
We can go further by refactoring our code with the help of **generics** and **reflection** in the benefit of reusability. To prevent creating a new adapter for each type it might be essential to follow this method on some occasions.

So we will implement another *TypeAdapter* because we need to handle the deserialization in a generic way:
{% highlight java %}
public class SingleAwareListTypeAdapter extends TypeAdapter<List> {

	private Gson gson;
	private TypeAdapter<?> objectTypeAdapter;
	private TypeAdapter<List> listTypeAdapter;

	public SingleAwareListTypeAdapter(Gson gson, Class elementClassType) {

		this.gson = gson;
		// we need to carry element's type by passing the element class type to object
		// type adapter otherwise gson deserialize our objects as LinkedTreeSet which we
		// do not expect that.
		this.objectTypeAdapter = gson.getAdapter(elementClassType);
		// list adapter for only serializing do not need the type of element inside list
		// here since all the output will be String at the end.
		this.listTypeAdapter = gson.getAdapter(List.class);
	}

	@Override
	public void write(JsonWriter out, List list) throws IOException {

		// Since we do not serialize our comment list with gson we can omit this part
		// but anyway we can simply implement by reusing gson list type adapter
		listTypeAdapter.write(out, list);
	}

	@Override
	public List read(JsonReader in) throws IOException {

		List deserializedObject = new ArrayList<>();

		// type of next token
		JsonToken peek = in.peek();

		// if the json field is single object just add this object to list as an
		// element
		if (JsonToken.BEGIN_OBJECT.equals(peek)) {
			deserializedObject.add(deserializeObject(in));
		}

		// if the json field is array then deserialize the objects inside
		if (JsonToken.BEGIN_ARRAY.equals(peek)) {
			in.beginArray();
			while (in.hasNext()) {
				if (JsonToken.BEGIN_OBJECT.equals(in.peek())) {
					deserializedObject.add(deserializeObject(in));
				}
			}
			in.endArray();
		}

		return deserializedObject;
	}

	private Object deserializeObject(JsonReader in) throws IOException {
		// just use gson object type adapter
		return objectTypeAdapter.read(in);
	}
}
{% endhighlight %}
Thus we can use the same adapter for different element types when we need this same behavior.

We should also create another *TypeAdapterFactory* implementation which will use our new generic *SingleAwareListTypeAdapter*:
{% highlight java %}
public class SingleAwareListTypeAdapterFactory implements TypeAdapterFactory {

	@Override
	public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {

		// proceed only if the incoming type is a parameterized type
		if (!ParameterizedType.class.isInstance(type.getType())) {
			return null;
		}

		ParameterizedType parameterizedType = ParameterizedType.class.cast(type.getType());
		Type elementType = parameterizedType.getActualTypeArguments()[0];
		Class rawElementType = Class.class.cast(elementType);

		// and proceed only if the element type is CommentModel
		if (!rawElementType.equals(CommentModel.class)) {
			return null;
		}

		return (TypeAdapter<T>) new SingleAwareListTypeAdapter(gson, rawElementType);
	}

}
{% endhighlight %}
We have implemented our factory with the help of reflection. Since we need the runtime class of incoming elements, first we got the *ParameterizedType* then obtained the runtime *Class* type of the element inside the *Collection*. In this way, we passed the runtime type of the element into *SingleAwareListTypeAdapter* as a parameter in the constructor. It is important to let the Gson know the element's type information in runtime otherwise our comment models are deserialized as *LinkedTreeSet* by default and we do not expect that.

Last, let's change our *ArticleModel* to use our new generic adapter factory:
{% highlight java %}

@JsonAdapter(SingleAwareListTypeAdapterFactory.class)
@Expose
private List<CommentModel> comments;

{% endhighlight %}
As a result, with the help of **generics** and **reflection**, we can use the same adapter, *SingleAwareListTypeAdapterFactory*, for the other properties with different element types as well. Consequently this will lead us to our goal and provide an effective way of **reusability**.

### 9. Conclusion
In this tutorial, **we learned how to implement custom adapters to handle variable responses in Gson**.

The source code of examples shown in this tutorial are available [over on GitHub](https://github.com).
