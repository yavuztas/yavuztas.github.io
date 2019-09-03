---
layout: post
title:  "How to call default serializer from custom serializer in Jackson"
date:   2014-11-30 14:34:25
categories: java jackson
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---

## 1. Introduction
In this short tutorial, we will show how to access default serializers inside a custom json serializer in the [Jackson Library](https://search.maven.org/search?q=a:jackson-databind%20AND%20g:com.fasterxml.jackson.core).

## 2. Custom Serializers in Jackson
On some occasions, serializing our data structure in an exact one-on-one way may not be the appropriate or simply is not what we want. We may want to customize on purpose with an extended or simplified view of our data.

So, this is where the custom serializers of the Jackson come into play.

Let's have a look our sample *Folder* class which we want to serialize:
{% highlight java %}
public class Folder {

	private Long id;

	private String name;

	private String owner;

	private Date created;

	private Date modified;

	private Date lastAccess;

	private List<File> files = new ArrayList<>();

	// standard getters and setters

}
{% endhighlight %}

And the *File* class which is defined as a *List* inside our *Folder* class:
{% highlight java %}
public class File {

	private Long id;

	private String name;

	// standard getters and setters

}
{% endhighlight %}

Let's imagine we want a simplified view of our *Folder* class which only the *name* field would be enough as our need.

So, let's create a custom serializer for our *Folder* class to achieve this purpose:
{% highlight java %}
public class FolderJsonSerializer extends StdSerializer<Folder> {

	public FolderJsonSerializer() {
		super(Folder.class);
	}

	@Override
	public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {

		gen.writeStartObject();
		gen.writeStringField("name", value.getName());

		gen.writeArrayFieldStart("files");
		for (File file : value.getFiles()) {
			gen.writeStartObject();
			gen.writeNumberField("id", file.getId());
			gen.writeStringField("name", file.getName());
			gen.writeEndObject();
		}
		gen.writeEndArray();

		gen.writeEndObject();

	}

}
{% endhighlight %}

Thus, we can serialize *Folder* class without the whole data but only a specific view of it:
{% highlight javascript %}
{
	"name": "Root Folder",
	"files": [
		{"id": 1, "name": "File 1"},
		{"id": 2, "name": "File 2"}
	]
}
{% endhighlight %}

## 3. Using Internal ObjectMapper
The main advantage of using custom serializers is, we do not have to modify our class structure and we can easily decouple our expected behavior from class itself.

Although custom serializers provide us flexibility that we can alter every properties in a detailed way, we may also improve it by reusing Jackson's default serializers.

One way of using the default serializers is to access the internal *ObjectMapper* class:
{% highlight java %}
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {

	gen.writeStartObject();
	gen.writeStringField("name", value.getName());

	ObjectMapper mapper = (ObjectMapper) gen.getCodec();
	gen.writeFieldName("files");
	String stringValue = mapper.writeValueAsString(value.getFiles());
	gen.writeRawValue(stringValue);

	gen.writeEndObject();

}
{% endhighlight %}

So, the Jackson simply handles the heavy lift, serializes the *List* of *File* classes, then our output will be the same:
{% highlight javascript %}
{
	"name": "Root Folder",
	"files": [
		{"id": 1, "name": "File 1"},
		{"id": 2, "name": "File 2"}
	]
}
{% endhighlight %}

## 4. Using SerializerProvider
Another way of calling the default serializers is using the *SerializerProvider* and delegating the process to the default serializer of the type *File*.

Let's simplify our code a little bit by the help of *SerializerProvider*:
{% highlight java %}
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {

	gen.writeStartObject();
	gen.writeStringField("name", value.getName());

	provider.defaultSerializeField("files", value.getFiles(), gen);

	gen.writeEndObject();

}
{% endhighlight %}

Similarly, we get the same output:
{% highlight javascript %}
{
	"name": "Root Folder",
	"files": [
		{"id": 1, "name": "File 1"},
		{"id": 2, "name": "File 2"}
	]
}
{% endhighlight %}

## 5. A Possible Recursion Problem
Depends on use cases, we may need to extend our serialized data by including more details for *Folder*. This might be for a legacy system or an external application to be integrated that we do not have a chance to modify.

Let's change our serializer to create another field as *details* for our serialized data to simply expose all the fields of *Folder* class:
{% highlight java %}
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {

	gen.writeStartObject();
	gen.writeStringField("name", value.getName());

	provider.defaultSerializeField("files", value.getFiles(), gen);

    // this line causes exception
	provider.defaultSerializeField("details", value, gen);

	gen.writeEndObject();

}
{% endhighlight %}

This time we get a *StackOverflowError* exception.

When we define a custom serializer, Jackson internally overrides the original *BeanSerializer* instance which is created for the type *Folder*. Consequently, our *SerializerProvider* finds the customized serializer every time, instead of the default one and this causes an infinite loop.

## 6. Using BeanSerializerModifier
A possible workaround to store the default serializer for the type *Folder* before the Jackson internally overrides it.

Let's modify our serializer and add an extra field as *defaultSerializer*:
{% highlight java %}
private final JsonSerializer<Object> defaultSerializer;

public FolderJsonSerializer(JsonSerializer<Object> defaultSerializer) {
	super(Folder.class);
	this.defaultSerializer = defaultSerializer;
}
{% endhighlight %}

Next, create an implementation of *BeanSerializerModifier* to pass the default serializer:
{% highlight java %}
public class FolderBeanSerializerModifier extends BeanSerializerModifier {

	@Override
	public JsonSerializer<?> modifySerializer(SerializationConfig config, BeanDescription beanDesc, JsonSerializer<?> serializer) {

		if (beanDesc.getBeanClass().equals(Folder.class)) {
			return new FolderJsonSerializer((JsonSerializer<Object>) serializer);
		}

		return serializer;
	}

}
{% endhighlight %}

Besides, we need to register this *BeanSerializerModifier* as a module in order to make it work:
{% highlight java %}
ObjectMapper mapper = new ObjectMapper();

SimpleModule module = new SimpleModule();
module.setSerializerModifier(new FolderBeanSerializerModifier());

mapper.registerModule(module);
{% endhighlight %}

Then, we use the *defaultSerializer* for the *details* field:
{% highlight java %}
@Override
public void serialize(Folder value, JsonGenerator gen, SerializerProvider provider) throws IOException {

	gen.writeStartObject();
	gen.writeStringField("name", value.getName());

	provider.defaultSerializeField("files", value.getFiles(), gen);

    gen.writeFieldName("details");
    defaultSerializer.serialize(value, gen, provider);

	gen.writeEndObject();

}
{% endhighlight %}

Last, we may want to remove *files* field from the *details* since we already write it into the serialized data separately.

So, we simply ignore the *files* field in our *Folder* class:
{% highlight java %}
@JsonIgnore
private List<File> files = new ArrayList<>();
{% endhighlight %}

Finally, the problem solved and we get our expected output as well:
{% highlight javascript %}
{
	"name": "Root Folder",
	"files": [
		{"id": 1, "name": "File 1"},
		{"id": 2, "name": "File 2"}
	],
	"details": {
		"id":1,
		"name": "Root Folder",
		"owner": "root",
		"created": 1565203657164,
		"modified": 1565203657164,
		"lastAccess": 1565203657164
	}
}
{% endhighlight %}

## 7. Conclusion
In this tutorial, **we learned how to call default serializers inside a custom serializer in Jackson Library**.

Like always, all the code examples used in this tutorial are available [over on GitHub](https://github.com).
