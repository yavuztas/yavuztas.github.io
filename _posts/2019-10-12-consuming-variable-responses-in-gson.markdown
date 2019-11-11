---
layout: post
title:  "Consuming Variable Responses in Gson"
date:   2019-10-12 18:30:00
readtime: 5 min read
categories: java gson
tags: featured
image: /assets/article_images/2019-10-12-consuming-variable-responses-in-gson/variable-responses-gson.jpg
image2: /assets/article_images/2019-10-12-consuming-variable-responses-in-gson/variable-responses-gson-mobile.jpg
excerpt: No matter being public or private, <b>Restful APIs</b> are the most popular way to integrate our applications with the world outside. This means you do not have a chance to alter the services you consume, instead, you should...
citation: Photo by <a href="https://unsplash.com/@abrahambarrera?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Abraham Barrera</a> on <a href="https://unsplash.com/s/photos/multi-value?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
---
No matter being public or private, **Restful APIs** are the most popular way to integrate our applications with the world outside. This means you do not have a chance to alter the services you consume, instead, you should adapt to them most of the time.

Since Java is a static-typed language, it can be a challenge while you are consuming and mapping the data into your model objects, especially if the response data changes depending on the use case.

Today, we will show how to handle variable response structures with the [**Gson library**](https://code.google.com/p/google-gson/).

For the basic usage of **Gson** you may consider reading [gson serialization guide](https://www.baeldung.com/gson-serialization-guide) and [gson deserialization guide](https://www.baeldung.com/gson-deserialization-guide) before a deep dive.

### Maven Dependency
Before we start, we need to add the Gson dependency into our project's *pom.xml:*
{% highlight xml %}
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.8.5</version>
</dependency>
{% endhighlight %}
### Sample Response
Let's pick a sample API for a blog which services detailed information about its articles along with the comments which may be more than one:
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
However, if there is only one single comment then its structure changes into object hash form:
{% highlight json %}
{
    "id": 1,
    "name": "sample article",
    "comments": {"id":1, "text":"some comment text"}
}
{% endhighlight %}
### Writing a Response Model
In order to map the data, we need a POJO as a response model. Let's start by defining a simple class that matches the main response structure:
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
### Consuming the Response
Since we use *List&lt;CommentModel&gt;* for the *comments* field, we can easily map multiple values which Gson already does the heavy lifting for us:
{% highlight java %}
String responseText = "{
  "id": 1,
  "name": "sample article",
  "comments": [
    {"id":1, "text":"some comment"},
    {"id":2, "text":"some another comment"}
  ]}";
Gson gson = new Gson();
ArticleModel model = gson.fromJson(responseText, ArticleModel.class);

Assert.assertEquals(1, model.getId().longValue());
Assert.assertEquals("sample article", model.getName());
Assert.assertEquals(2, model.getComments().size());
Assert.assertEquals(1, model.getComments().get(0).getId().longValue());
Assert.assertEquals("some comment", model.getComments().get(0).getText());
Assert.assertEquals(2, model.getComments().get(1).getId().longValue());
Assert.assertEquals("some another comment", model.getComments().get(1).getText());
{% endhighlight %}
Although this configuration works for the multiple values of comments, if it changes to a single value then we need to define a custom *TypeAdapter* in order to handle both single and multiple values.
### Custom *TypeAdapter* in Gson
In order to achieve a custom JSON deserialization behavior, we need to create a *TypeAdapter*. This behavior will be responsible for adding any single value to the list as the same as multiple values are being added automatically by Gson.

For the beginning, let's create a Gson *TypeAdapter* for this purpose:
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
We need to access *Gson* instance inside our *TypeAdapter* to preserve and reuse default deserialization behavior for *Object* and *Collection* types.

So, let's define a constructor and fields of *Gson* instance and some default adapters:
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
Since we aim to deserialize only, we can omit the *write* method of our adapter. However, we can implement in any case by simply delegating it to the default adapters.

Next, let's implement *write* method by using *listTypeAdapter*:
{% highlight java %}
@Override
public void write(JsonWriter out, List list) throws IOException {

    listTypeAdapter.write(out, list);

}
{% endhighlight %}
Then, we implement the *read* method of our adapter to come up with our expected deserialization behavior:
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
### *TypeAdapterFactory* in Gson
We may consider using *TypeAdapterFactory* in Gson. There are two benefits of this; one of them is we can implement a generic factory method to create different *TypeAdapter* classes for different types. The other one is **we can access the underlying *Gson* instance and reuse it** which we are primarily looking for now.  

So, let's create a custom *TypeAdapterFactory*:
{% highlight java %}
public class CommentListTypeAdapterFactory implements TypeAdapterFactory {

	@Override
	public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {

		return (TypeAdapter<T>) new CommentListTypeAdapter(gson);
	}

}
{% endhighlight %}
As we can see, we delegated the existing *Gson* instance to our *CommentListTypeAdapter* by the help of our custom *TypeAdapterFactory*.
### Registering *TypeAdapterFactory* into Gson
Finally, we need to register our custom *TypeAdapterFactory* into the *Gson* context to make it work.

Let's update our *ArticleModel*:
{% highlight java %}

@JsonAdapter(CommentListTypeAdapterFactory.class)
@Expose
private List<CommentModel> comments;

{% endhighlight %}
Now we can consume single value structures like multiple values with the same *ArticleModel*:
{% highlight java %}

String responseText = "{
  "id": 1,
  "name": "sample article",
  "comments": {"id":1, "text":"some comment"}
  }";
Gson gson = new Gson();
ArticleModel model = gson.fromJson(responseText, ArticleModel.class);

Assert.assertEquals(1, model.getId().longValue());
Assert.assertEquals("sample article", model.getName());

Assert.assertEquals(1, model.getComments().size());
Assert.assertEquals(1, model.getComments().get(0).getId().longValue());
Assert.assertEquals("some comment", model.getComments().get(0).getText());

{% endhighlight %}
### Further Improvements With the Power of Generics
We can go further by refactoring our code with the help of **generics** and **reflection** in the benefit of reusability. To prevent creating a new adapter for each type it might be essential to follow this method on some occasions.

Similarly, we implement another *TypeAdapter* because we need to handle it in a generic way:
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
Thus, we can use the same adapter for different element types when we need the same behavior.

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

		return (TypeAdapter<T>) new SingleAwareListTypeAdapter(gson, rawElementType);
	}

}
{% endhighlight %}
It is important to **let the Gson know the list element's type information in runtime otherwise our models are deserialized as *LinkedTreeSet* by default.**

Since we need the runtime class of incoming list elements, first we obtained the *ParameterizedType*, then the runtime *Class* type of the element inside the *Collection*.

In this way, we passed the type information of the element into the *SingleAwareListTypeAdapter* as a parameter in the constructor.

Finally, let's change our *ArticleModel* to use our new generic adapter factory:
{% highlight java %}

@JsonAdapter(SingleAwareListTypeAdapterFactory.class)
@Expose
private List<CommentModel> comments;

{% endhighlight %}
As a result, with the help of **generics** and **reflection**, we can use the same adapter, *SingleAwareListTypeAdapterFactory*, for the other properties with different element types as well.

Consequently, this will provide an effective way of **reusability** and lead us to our goal.

### Conclusion
In this tutorial, **we learned how to implement custom adapters to handle variable responses in Gson**.

All the source code of examples shown in this tutorial are available [over on GitHub](https://github.com/yavuztas/java-gson-multivalue).
