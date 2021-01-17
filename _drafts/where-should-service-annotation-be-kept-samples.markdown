---
layout: post
title:  "Where should @Service annotation be kept"
date:   2014-11-30 14:34:25
categories: java spring
tags: regular
image: /assets/article_images/java-quiz-questions/java-quiz.jpg
image2: /assets/article_images/java-quiz-questions/java-quiz-mobile.jpg
---
## 1. Introduction
- Tell something about there is a debate about where the Spring's @Service annotation should be used, interface or concerete classes, among developers.

Searching for the right solution, good, better and hopefully the best, in the guidance of requirements is what developers always do. Naturally, there are debates sometimes.
Especially, in the age of modern and capable frameworks, it can be quite often.

One of them is the usage of the Spring's @Service annotation. Since Spring provides some alternative ways to define beans, sometimes it worths to pay attention the whereabouts of stereotype annotations.

- it can be hard to decide in terms of keeping the codebase maintainable and viable at the same time.

In this tutorial, we will break down the @Service annotation usage and cover all the possible ways. We will also give real life examples - to explain when and why we can use each of them.


## 4. Why would I use @Service on Concerete Classes?
- We keep our interfaces clean and decoupled if we choose to annotate implementations/concerete classes.

Think about that our company has formed by financial experts and we developed two successful stock market strategies, to automate buying and selling stocks and make profit. We decided to open these two predefined strategies for our beta users and test how they work.

Here is the design to implement our Stock Market Signal Application as an example:
{% highlight java %}
public interface TradeStrategy {

    boolean hasBuySignal(String stock, Double[] hourlyClosePrices);

}
{% endhighlight %}

One of our strategy implementations is *BollingerBandsStrategy* and we define it as a Spring Bean:
{% highlight java %}
@Service("strategy1")
public BollingerBandsStrategy implements TradeStrategy {

    public boolean hasBuySignal(String stock, Double[] hourlyClosePrices){
        //...
    }

}
{% endhighlight %}

Next, we define our other strategy implementation, *MovingAverageStrategy*, which is also another Spring Bean:
{% highlight java %}
@Service("strategy2")
public MovingAverageStrategy implements TradeStrategy {

    public boolean hasBuySignal(String stock, Double[] hourlyClosePrices){
        //...
    }

}
{% endhighlight %}

Finally, we use our concerete strategies to produce signals and inform our users:
{% highlight java %}
@Component
public class SignalEmitter {

    static final String HOURLY_CRON_EXPRESSION = "0 0 * * * * ";

    @Autowired
    StockService stockService;

    @Autowired
    TradeStrategy strategy1;

    @Autowired
    TradeStrategy strategy2;

	@Scheduled(cron = HOURLY_CRON_EXPRESSION)
	public void emit() {

        Map<String, Double[]> data = stockService.getDataForStocks();
        for(String stock: data.keys()){
            boolean signalForStrategy1 = strategy1.hasBuySignal(stock, data.get(stock));
            if(signalForStrategy1){
                emitSignalForStock(stock);
            }
            boolean signalForStrategy2 = strategy2.hasBuySignal(stock, data.get(stock));
            if(signalForStrategy2){
                emitSignalForStock(stock);
            }
        }

	}

    void emitSignalForStock(String stock){
        //...
    }

}
{% endhighlight %}


## 5. Why would I use @Service on Interfaces?
- In some rare cases, if we want to define all implementations of a single interface as Spring beans automatically then we need to know some Spring internals (like how Spring names its beans by default, etc.) For example, for dynamically created Spring beans in runtime we can consider annotating the interface.

- Con side is that we are coupling our interface with a specific framework i.e. Spring, by using spring specific annotation. As interfaces are supposed to be decoupled from implementation, we should not use any framework specific materials in the interfaces.

- We can mention about Spring-Data-JPA Repository interfaces which are exceptional cases.

Now, let's refactor our stock market signal application. Later, we have implemented some more strategies and these strategies are likely to be more in time. Also, we decided to provide for our users to select their strategies and use them to get signals.

To refactor our application, we made a decision. We remove *@Service* annotations from the implementations and we annotate only our interface *TradeStrategy* in order to define every implementation as a Spring Bean automatically:
{% highlight java %}
@Service
public interface TradeStrategy {

    boolean hasBuySignal(String stock, Double[] hourlyClosePrices);

}
{% endhighlight %}

At the end our *SignalEmitter* is going to be:
{% highlight java %}
@Component
public class SignalEmitter {

    static final String HOURLY_CRON_EXPRESSION = "0 0 * * * * ";

    @Autowired
    StockService stockService;

    @Autowired
    UserService userService;

    @Autowired
    ApplicationContext applicationContext;

	@Scheduled(cron = HOURLY_CRON_EXPRESSION)
	public void emit() {

        User user = userService.getAuthUser();
        String strategyName = user.getStrategyName();
        TradeStrategy userStrategy = applicationContext.getBean(strategyName, TradeStrategy.class);

        Map<String, Double[]> data = stockService.getDataForStocks();
        for(String stock: data.keys()){
            boolean signalForStrategy1 = userStrategy.hasBuySignal(stock, data.get(stock));
            if(signalForStrategy1){
                emitSignalForStock(stock);
            }
        }

	}

    void emitSignalForStock(String stock){
        //...
    }

}
{% endhighlight %}

## 6. Conclusion
Keeping @Service annotation on implementations:
- We can keep our interfaces clean and decoupled if we choose to annotate implementations.
- It's easier to configure each bean's name by using the @Service annotation on them.

Keeping @Service annotation on interfaces:
- We need to know a little more implicit Spring internals (like how Spring define names for beans) to configure it correctly.
- We are coupling our interface with a specific framework i.e. Spring, by using spring specific annotation. As interfaces are supposed to be decoupled from implementation.

And make a suggestion to which way to use.

In my opinion, it would be annotating the implementations in most cases by the way.
