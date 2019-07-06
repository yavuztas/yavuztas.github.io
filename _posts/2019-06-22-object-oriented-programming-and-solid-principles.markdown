---
layout: post
title:  "Object Oriented Programming and SOLID Principles"
date:   2019-06-22 16:33:00
categories: java oop
image: "/assets/article_images/2019-06-22-object-oriented-programming-and-solid-principles/header-java.png"
image2: "/assets/article_images/2019-06-22-object-oriented-programming-and-solid-principles/header-java-mobile.png"
---
One of the popular programming paradigms is called **Object Oriented Programming**, generally abbreviated as OOP, suggests that **objects** should be used in computer programs. Objects are special structures in programming contain **data** in forms of their properties, also named as attributes. Besides they contain procedures that are responsible for altering this data. These are mostly called as functions.

Happily, Java is a clean example for the OOP languages and contains many high-level structures to for developers which are formed by objects. Thus we do not need to worry about most of the low-level operations. However, there are further principles to learn in order to apply OOP correctly. These are widely known as solid principles in the programming world. When we say solid, this is because they literally form the **SOLID**. It is a funny story about that our former computer scientists culminated with an acronym for five OOP design principles intended to make software designs better and understandable.

#### What are these SOLID principles?
We have five principles each of them stands for each letter **S-O-L-I-D** which are *[Single Responsibility Principle](#single-responsibility-principle), [Open Closed Principle](#open-closed-principle), [Liskov Substitution Principle](#liskov-substitution-principle), [Interface Segregation Principle](#interface-segregation-principle) and [Dependency Inversion Principle](#dependency-inversion-principle).*

#### Single Responsibility Principle
Single Responsibility Principle in software programming suggests that one should have only one single responsibility. We usually refer any module, class or function here. Hence from the OOP perspective, it mostly refers to objects which can be modules or classes. In this way, one object can only modify one part of the software's specifications. Objects trying to handle more than one responsibility will eventually ensue fragility and become impossible to maintain. Thus it is clearly seen that violation of this principle causes us the famous [God Object](https://en.wikipedia.org/wiki/God_object){:target="blank"} anti-pattern in time.

An example to indicate a violation of the single responsibility principle:

>LoginManager.java
{:.filename}

{% highlight java %}
if(LoginType.LOCAL_DB.equals(type)){

    // authenticating user from local db

} else if(LoginType.REMOTE_DB.equals(type)){

    // authenticating user from remote db

} else if(LoginType.LDAP.equals(type)){

    // authenticating user from ldap

} else if(LoginType.SOCIAL.equals(type)){

    // authenticating user from social network accounts

}
// and this conditional cases can go on...
{% endhighlight %}

So this code seems to start smelling like becoming a **god object**. Although it looks like a semi-god for now, it definitely intends to become bigger in time unless we do not stop it by **refactoring** like:

>LoginManager.java
{:.filename}

{% highlight java %}
public LoginManager() {
    // registering our manager implementations
    loginManagers.add(new LocalDBLoginManager());
    loginManagers.add(new RemoteDBLoginManager());
    loginManagers.add(new LdapLoginManager());
    loginManagers.add(new SocialLoginManager());
}

public void authenticate(User user, LoginType type) {
    // getManager method returns the right implementation
    // according to login type we need
    ILoginManager loginManager = getManager(type);
    loginManager.authenticate(user);
}
{% endhighlight %}

You can check the source code for this sample over [here](https://github.com/yavuztas/java-solid-principles/blob/master/src/main/java/dev/yavuztas/samples/solid/srp/LoginManager.java){:target="blank"}. Besides you can read [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle){:target="blank"} over on Wikipedia for further details.

#### Open Closed Principle
Open Closed Principle in software programming simply means that an ideal software application should be open for extensions but closed for modifications. Doing **modification** here is thought for changing the existing codes of pre-made modules, classes, etc. On the other hand, what is mentioned when we say **extension** is adding new classes, modules or even functions without touching the rest of the codebase.

Some implications of modification:
* Increases fragility, decreases maintainability.
* Causes strictly tight modules and classes.
* Leads to unexpected bugs, especially in projects which do not have enough test coverage.

A clear example to show a piece of code which will probably need modifications later on:

>ModificationManager.java
{:.filename}

{% highlight java %}
if(ModificationType.LESS.equals(type)){

    // less modification :)

} else if(ModificationType.MEDIUM.equals(type)){

    // medium modification :(

} else if(ModificationType.QUITE.equals(type)){

    // quite a lot modification :O

} else if(ModificationType.ENORMOUS.equals(type)){

    // vast modification >:O

}
// and this conditional cases can go on...
{% endhighlight %}

One proper solution to this problem is **Chain of Responsibility** pattern which is also a good example of design by extensions, can be achieved like the code below:

>ModificationManager.java
{:.filename}

{% highlight java %}
public ModificationManager() {
    // new modifier implementations can be easily added
    // without touching the rest of the code
    modifiers.add(new LessModifier());
    modifiers.add(new MediumModifier());
    modifiers.add(new QuiteModifier());
    modifiers.add(new EnormousModifier());
}

// main part of the code that handles the logic
// is not affected by adding or removing different type of modifiers
public boolean modifySystem(ModificationType type) {

    for (IModifier modifier : modifiers) {
        // each modifier instance knows that they can modify it
        // or not and returns the result as boolean
        if (modifier.modify(type)) {
            return true;
        }
    }

    // if we are here then it seems no modification is done
    return false;

}

{% endhighlight %}

You can check the source code for this sample over [here](https://github.com/yavuztas/java-solid-principles/blob/master/src/main/java/dev/yavuztas/samples/solid/ocp/ModificationManager.java){:target="blank"}.

#### How the OCP can be used in Java?
One of the best practices is **programming to interface** when it comes to applying OCP into java. Programming to interface means to prefer using interfaces instead of concrete types unless you do not specifically need to. Interfaces are the contracts to expose the behavior type of our program to the outer world. With the help of well-defined interfaces, you always have a chance to create new implementations and easily extend your project without affecting the world outside, which technically means adding an extension. Hence we can say that interfaces really play nice with the OCP.

A simple example to show an advantage of using interfaces over concrete types:

>CoffeeMachine.java
{:.filename}

{% highlight java %}
public interface Coffee {
    public void taste();
}

Coffee coffee = new FilterCoffee();

// you taste filter coffee!
coffee.taste();

// After some time we discover a new taste and add our new coffee formula!
Coffee coffee = new EspressoCoffee();

// now you taste espresso, we do not need to modify the code below!
coffee.taste();
{% endhighlight %}

Besides you can read [open closed principle](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle) over on Wikipedia for further details.
#### Liskov Substitution Principle
Liskov Substitution Principle suggests that objects in a software program should be replaceable with the instances of their subtypes without need to change properties of theirs. Another use case of interfaces transpires here since we need a behavioral similarity between subtypes, also called as **strong behavioral subtyping**. Different behaviors can output different results so we need to group subtypes with similar behavior by using interfaces not to break our program's communication with the outside.

An example to demonstrate this problem:

{% highlight java %}
public class Fish {
    public void swim(){
        System.out.println("I'm swimming")
    }
}

public class DeadFish extends Fish {
    public void swim(){
        System.out.println("Cannot swim because I'm dead!")
    }
}

// Let's say that we need a fishing pool which every Fish instance should swim.
// However as you can see some instances will not be able to swim because they are dead.
List<Fish> pool = new ArrayList<>();
pool.add(new Fish());
pool.add(new Fish());
pool.add(new DeadFish());

for(Fish fish:pool){
    fish.swim();
}
{% endhighlight %}

An elegant solution comes with the help of interfaces to discriminate subtypes according to their behaviors:

{% highlight java %}
public interface Alive {
    public void swim();
}

public interface Dead {
    public void sink();
}

public class AliveFish extends Fish implements Alive {

    @Override
    public void swim() {
        System.out.println("I'm swimming because I'm alive :)");
    }
}

public class DeadFish extends Fish implements Dead {

    @Override
    public void swim() {
        System.out.println("Cannot swim because I'm dead!");
    }

    public void sink() {
        System.out.println("I'm sinking :(");
    }
}

// So we need only alive fish for our fishing pool.
// Now we are sure that every Fish instance in our pool can swim!
List<Alive> pool = new ArrayList<>();
pool.add(new AliveFish());
pool.add(new AliveFish());
pool.add(new AliveFish());
// compilation error prevents us to add DeadFish instances by mistake!
// pool.add(new DeadFish());

for(Alive fish:pool){
    fish.swim();
}
{% endhighlight %}

You can check the source code for this sample over [here](https://github.com/yavuztas/java-solid-principles/blob/master/src/main/java/dev/yavuztas/samples/solid/lsp/FishingPool.java){:target="blank"}. Besides you can read [liskov substitution principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle){:target="blank"} over on Wikipedia for further details.

#### Interface Segregation Principle
Interface Segregation Principle in software simply tells us that instead of one general-purpose interface, it is better to use many client-specific ones. One obvious problem we can encounter when we violate this principle, is the boilerplate invasion of meaningless, empty methods.

Let us show this problem with an example:

{% highlight java %}
public interface Animal {

    public void swim();
    public void fly();
    public void run();

}

public class SwimmingAnimal implements Animal {

    public void swim(){
        System.out.println("Oh it's swimming, I know how to swim :)");
    }

    // Ideally we do not need to implement the methods below
    public void fly(){
        System.out.println("What am i going to do?");
    }

    public void run(){
        System.out.println("What am i going to do?");
    }

}
{% endhighlight %}

And the solution would be splitting our general interface into more specific ones such as:

{% highlight java %}
public interface Animal {
    // let this be just a marker interface
}

public interface CanSwim extends Animal {
    public void swim();
}

public interface CanFly extends Animal {
    public void fly();
}

public interface CanRun extends Animal {
    public void run();
}

// no need to implement unnecessary methods
public class SwimmingAnimal implements CanSwim {
    public void swim(){
        System.out.println("Oh it's swimming, I know how to swim :)");
    }
}
{% endhighlight %}

You can check the source code for this sample over [here](https://github.com/yavuztas/java-solid-principles/blob/master/src/main/java/dev/yavuztas/samples/solid/isp/AnimalFarm.java){:target="blank"}. Besides you can read [interface segregation principle](https://en.wikipedia.org/wiki/Interface_segregation_principle){:target="blank"} over on Wikipedia for further details.

#### Dependency Inversion Principle
Dependency Inversion Principle states that in a software program high-level objects should not depend on low-level objects, on the contrary, both should depend on abstractions. Not unlike, concrete classes should depend on abstractions, not vice versa. After these abstract explanations let us be a little bit more explanatory.

An example of a DIP violation:

{% highlight java %}
public class OperatingSystem {

    private HttpService httpService = new HttpService();
    private SmtpService smtpService = new SmtpService();

    public void runOnStartup() {
        httpService.startHttpd();
        smtpService.startSmtpd();
    }

}
{% endhighlight %}

Instead of depending on concrete classes we should definitely make an abstraction by the help of interfaces and refactor our tiny operating system to accept only abstract services in order to initiate on the os startup.

See the code below:
{% highlight java %}
public class HttpService implements IService {
    public void start(){
        System.out.println("Starting httpd service...");
    }
}

public class SmtpService implements IService {
    public void start(){
        System.out.println("Starting smtpd service...");
    }
}

public class OperatingSystem {

    private List<IService> services = new ArrayList<>();

    public OperatingSystem() {
        register(new HttpService());
        register(new SmtpService());
    }

    public void register(IService service){
        this.services.add(service);
    }

    public void runOnStartup() {
        this.services.forEach(s -> s.start());
    }

}
{% endhighlight %}

You can check the source code for this sample over [here](https://github.com/yavuztas/java-solid-principles/blob/master/src/main/java/dev/yavuztas/samples/solid/dip/OperatingSystem.java){:target="blank"}. Besides you can read [dependency inversion principle](https://en.wikipedia.org/wiki/Dependency_inversion_principle){:target="blank"} over on Wikipedia for further details.

After we explained OOP and the solid principles shortly we will go on *containers, inversion of control and dependency injection* and give some examples about how they are applied in our [next post]({{ site.url }}/java/oop/2019/07/04/continers-inversion-of-control-and-dependency-injection.html).
