---
layout: post
title:  "Type Based Global Events in Vue.js"
date:   2019-11-10 22:30:00
categories: javascript vuejs
tags: regular
image: /assets/article_images/2019-11-10-type-based-global-events-in-vuejs/type-based-global-events-in-vuejs.jpg
image2: /assets/article_images/2019-11-10-type-based-global-events-in-vuejs/type-based-global-events-in-vuejs-mobile.jpg
excerpt: In one of the latest freelance projects of mine, my client prefers <b>Vue Framework</b> which is recently super popular on the frontend side.
citation: Photo by <a href="https://unsplash.com/@mathyaskurmann?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Mathyas Kurmann</a> on <a href="https://unsplash.com/s/photos/post?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
---
In one of the latest freelance projects of mine, my client prefers [Vue Framework](https://vuejs.org/v2/guide/) which is recently super popular on the frontend side.

So, I dived into **Vue**. I can say that it is very practical and effective on the very first sight.

Besides, when we compare it with other predominant competitors like Angular and Aurelia, **we can easily notice Vue has a very slight learning curve.**

However, it didn't take so long for me to stumble on a bad feeling that my code is getting unmanageable and becoming unable to be followed.

Undoubtfully, it wasn't a big surprise to me because **this is mostly what dynamic-typed languages make us resent the harsh trade-off along with their super cool advantages.**

Today, I am going to show a practical and effective way of using global events in **Vue Framework.**

## A Simple Event Bus in Vue
The typical way of implementing a global event bus in Vue is just using the *Vue* object itself:
{% highlight javascript %}
// create a Vue instance somewhere you can access globally
let eventBus = new Vue()

// register an event handler
eventBus.$on("onAppStarted", () => console.log("App Started!"))

// publish an event
eventBus.$emit("onAppStarted")

// Super easy, right?
{% endhighlight %}

## The Problem Coming From Strings
As long as our application has more than a couple of lines, sooner or later, we start stressing to follow which components publish and which others listen to them.

Therefore, we can imagine how hard to identify a simple typo like *onApStarted* instead of *onAppStarted* as an event name in a large  project:
{% highlight javascript %}
eventBus.$on("onApStarted", () => {
  // some business logic here  
})
{% endhighlight %}

## Implicit Event Parameters
Moreover, because we don't define any corresponding type or interface for our event parameters **only God knows what and how many parameters might be for the *onAppStarted* event.**

In order to identify we resent doing this kind of tests everytime we confuse:
{% highlight javascript %}
eventBus.$on("onAppStarted", (...args) => {
  args.forEach(e => console.log(e))    
})
{% endhighlight %}

## A Proper Solution Comes From ES6+
As being a fan of static-typed Java world, whatever language I do, I prefer using types clearly unless it's super unconventional for the specific language.

Thus, I will show a solution to get rid of these string-based event names by using the capabilities **ECMAScript 6** and later offers.

### Defining Event Types
Let's create a separate file called *app-events.js* to define our **event types:**
{% highlight javascript %}
/**
* Event type to publish when app loads
* ProducedBy: components/preload.js
* ConsumedBy: App.vue, views/MainPanel.vue
**/
export class AppStartEvent {

  constructor(){
    // An event type with no arguments
  }

}

/**
* Event type to publish when code changes
* ProducedBy: views/CodePanel.vue
* ConsumedBy: views/MainPanel.vue
* @param {object} editor   editor instance
* @param {string} code     changed code value inside the editor
**/
export class CodeChangeEvent {

  constructor(editor, code){
    this.editor = editor
    this.code = code
  }

}
{% endhighlight %}
As we can notice, **defining event type classes and parameters in constructor explicitly offers us great readability.**

Although it is optional, we recommend keeping comments updated. This provides a way to follow the components which deal with a certain event type.

### Importing Event Types
When we want to use our events, we should import them into our components:
{% highlight javascript %}
import {AppStartEvent, CodeChangeEvent} from "@/app-events"
{% endhighlight %}
As we specify explicitly every event type we need, **it brings us another important benefit that we can easily identify what events are involved in a component.**

### Registering an Event
In order to register our event, simply we use our event types and their static *name* properties:
{% highlight javascript %}
import {AppStartEvent} from "@/app-events"

eventBus.$on(AppStartEvent.name, () => console.log("App Started!"))
{% endhighlight %}
Plus, we can expect the event type instance itself as a single argument instead of more than one arguments:
{% highlight javascript %}
import {AppStartEvent, CodeChangeEvent} from "@/app-events"

// we can access the event type instance as a single argument
eventBus.$on(AppStartEvent.name, event => console.log(event))

// also can access to event parameters
eventBus.$on(CodeChangeEvent.name, event => {
  console.log(event.editor)  
  console.log(event.code)  
})
{% endhighlight %}
### Publishing an Event
Now we can publish our events by creating a new instance of that event type:
{% highlight javascript %}
// no parameters
eventBus.$emit(AppStartEvent.name, new AppStartEvent())

// with parameters
eventBus.$emit(CodeChangeEvent.name, new CodeChangeEvent(editor, "some code here..."))
{% endhighlight %}

## Implementing a Wrapper Class
Certainly, we may proceed to define a class as *EventBus* and wrap the basic methods of *Vue* instance.
{% highlight javascript %}
class EventBus {

  $eventbus = new Vue()

  listen (eventClass, handler) {
    this.$eventBus.$on(eventClass.name, handler)
  }

  publish (event) {
    this.$eventBus.$emit(event.constructor.name, event)
  }

}
{% endhighlight %}
Therefore, we can use it in a more practical way:
{% highlight javascript %}
// register an event handler
EventBus.listen(AppStartEvent, () => console.log("App Started!"))

// publish an event
EventBus.publish(new AppStartEvent())
{% endhighlight %}

## Using as a Plugin
We may prefer to use our *EventBus* as a **Vue Plugin**:
{% highlight javascript %}
export default {

  $eventBus: null,

  install (Vue, options) {
    this.$eventBus = new Vue()
  },

  listen (eventClass, handler) {
    this.$eventBus.$on(eventClass.name, handler)
  },

  listenOnce (eventClass, handler) {
    this.$eventBus.$once(eventClass.name, handler)
  },

  remove (eventClass, handler) {
    if (handler) {
      this.$eventBus.$off(eventClass.name, handler)
    } else {
      this.$eventBus.$off(eventClass.name)
    }
  },

  removeAll () {
    this.$eventBus.$off()
  },

  publish (event) {
    this.$eventBus.$emit(event.constructor.name, event)
  }

}
{% endhighlight %}
In order to use the plugin, we should import and register to our *Vue* instance:
{% highlight javascript %}
import EventBus from '@/plugin/vue-event-bus';

Vue.use(EventBus)
{% endhighlight %}
Consequently, we can simply import and use in any other Vue component as well:
{% highlight javascript %}
import EventBus from '@/plugin/vue-event-bus';
import {AppStartEvent} from "@/app-events"

// register an event handler
EventBus.listen(AppStartEvent, () => console.log("App Started!"))

// publish an event
EventBus.publish(new AppStartEvent())
{% endhighlight %}

## Finally
In this short tutorial, **I explained how to implement type-based global events and use them in Vue**.

You can find the plugin [over on GitHub](https://gist.github.com/yavuztas/d1300752057de9314c8614d6a82ccc39).

So, what do you think about this approach or anything to extend? I would like to see your comments below!
