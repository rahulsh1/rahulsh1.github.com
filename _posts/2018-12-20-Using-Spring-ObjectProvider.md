---
layout: post
title: Using Spring's ObjectProvider
category: tech
tags: Spring Spring-Framework ObjectProvider
year: 2018
month: 12
day: 20
published: true
summary: Using Spring's ObjectProvider to retrieve a bean only if it actually exists or if a single candidate can be determined using programmatic resolution. With Iterable and Stream support added in 5.1, we can easily support cases (with the same code) when zero or more dependencies exist for the bean of a given type.
---

When Spring Framework 4.3 was released, it introduced [ObjectProvider](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/ObjectProvider.html). As Spring's blog [[1](#references)] mentions, this provided an extension to the existing [ObjectFactory](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/ObjectFactory.html) interface with handy signatures such as getIfAvailable, getIfUnique etc to retrieve a bean only if it actually exists or if a single candidate can be determined (such as a primary candidate in case of multiple matching beans). This also improved the programmatic resolution of dependencies.

With 5.0 release, this has been improved to support newer APIs that allows you to pass a `Supplier` for the getIfAvailable/getIfUnique APIs and a `Consumer` for the ifAvailable/ifUnique APIs.

And with 5.1 release, this has been further extended to give it `Iterable` and `Stream` support [[2](#references)].


## Diving into ObjectProvider
Let's take a look at the things you can do with ObjectProvider. We'll dive into the simple use-cases first and then cover the newer APIs too.

The code discussed below is available [here](https://github.com/rahulsh1/spring-objectprovider-examples).

### No dependencies
Let's say I've this simple Interface for a logging service as given below:
{% highlight java %}
public interface LogService {
  void log(String data);
}
{% endhighlight %}

I am going to use this LogService inside the Spring Bean as shown.

{% highlight java %}
@Component
public class ExampleOne {

  private final LogService logService;

  @Autowired
  public ExampleOne(LogService logService) {
    this.logService = logService;
  }

  public void runApps() {
    logService.log("some data");
  }
}
{% endhighlight %}

The above Spring bean wires in the LogService dependency using constructor-based injection.

You don't really need to mention `@Autowired` as part of the Constructor param as this is the only Constructor. This support came in with Spring 4.3 that allows `Implicit constructor injection for single-constructor` scenarios. I also don't need the `@Component` since I am explicitly registering it with the `ApplicationContext`. I am going to skip both in the later examples.

I'll now wire this up in the following test case and we shall see what happens when there are no dependencies for `LogService`.

{% highlight java %}
@Test(expected = UnsatisfiedDependencyException.class)
public void testExampleOneMissingBean() {
  try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
    context.register(ExampleOne.class);
    context.refresh();

    ExampleOne example = context.getBean(ExampleOne.class);
    example.runApps();
  }
}
{% endhighlight %}

Notice that no dependencies of `LogService` were registered with the ApplicationContext. There is only one User defined bean.
As expected, we get the `NoSuchBeanDefinitionException` for the field logService in `ExampleOne`.

> Throws Unsatisfied dependency expressed through field 'logService'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.example.beans.LogService' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

You may need to run your application in certain environments where it is normal for some dependencies to be missing (i.e. they are optional) and you don't want your application to blow up like above.

How do you go above handling such a case then?

### Making it Optional
Let’s see how the `ObjectProvider` can come to our rescue here to handle optional dependencies.

I'll update the type from `LogService` to `ObjectProvider<LogService>` and also modify the runApps method to use the `ifAvailable` API.

{% highlight java %}
public class ExampleTwo {

  private ObjectProvider<LogService> logService;

  public ExampleTwo(ObjectProvider<LogService> logService) {
    this.logService = logService;
  }

  public void runApps() {
    logService.ifAvailable(e -> e.log("some data"));
  }
}
{% endhighlight %}

Now when I run the same test as before, we don't get any exception and things are all good. No output is printed as expected.

Let's make sure this works even when a dependency actually exists. To do that, we register the `PlainLogger` bean as well with the ApplicationContext as part of `context.register(...)`.

{% highlight java %}
@Test
public void testExampleTwoWithOneLogger() {
  try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
    context.register(ExampleTwo.class, PlainLogger.class);  // Register the dependency
    context.refresh();

    ExampleTwo example = context.getBean(ExampleTwo.class);
    example.runApps();
  }
}

// The dependency...
public class PlainLogger implements LogService {
  @Override
  public void log(String data) {
    System.out.printf("Data [%s] at %d%n", data, System.currentTimeMillis());
  }
}
{% endhighlight %}

This time I get some output as expected:

    Data [some data] at 1545539958913

> Note:
You can also use the `java.util.Optional<T>` instead of `ObjectProvider<T>` but it works well only when zero/one implementation(s) exists.

### More than one dependency
Now what happens if there are more than one dependencies of `LogService`, which one would be used with the above? Any guesses?

I am registering two implementations here for `LogService`, namely `PlainLogger` and `JsonLogger`.

{% highlight java %}
@Test
public void testExampleTwoWithMultipleLoggers() {
  try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
    context.register(ExampleTwo.class, PlainLogger.class, JsonLogger.class);
    context.refresh();

    ExampleTwo example = context.getBean(ExampleTwo.class);
    example.runApps();
  }
}

// The two dependencies
public class PlainLogger implements LogService {
  @Override
  public void log(String data) {
    ...
  }
}

public class JsonLogger implements LogService {
  @Override
  public void log(String data) {
    System.out.printf("{\"log\": { \"message\": \"%s\", \"timestamp\": %d } }", data, System.currentTimeMillis());
  }
}
{% endhighlight %}

This would not work. Notice where this fails though. The wiring is fine though however when calling `runApps` Spring doesn't know which one to choose for you.

Since Spring cannot decide which specific implementation you want (and neither bean is marked as `@Primary` too), this would throw an exception.
> org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.example.service.LogService' available: expected single matching bean but found 2: plainLogger,jsonLogger

One fix for above would be to mark one of the implementations as `@Primary`. But what if you needed both the implementations?

To do that, I'll now use the API introduced in 5.1 for this (this is not supported pre-5.1 though).

We don't need to make changes to the type. The type remains the same, it is still `ObjectProvider<LogService>` and not `ObjectProvider<List<LogService>>`.

However the API to access the beans is a bit different. We use the `stream` API to access the dependencies. We could have used an enhanced for loop as well since the `ObjectProvider` interface extends `Iterable`. See the updated implementation of `runApps`

{% highlight java %}
public class ExampleThree {

  private ObjectProvider<LogService> logService;

  public ExampleThree(ObjectProvider<LogService> logService) {
    this.logService = logService;
  }

  public void runApps() {
    logService.stream().forEach(e -> e.log("some app data with " + getClass().getSimpleName()));
  }
}
{% endhighlight %}

Let's test this out.

{% highlight java %}
@Test
public void testExampleThreeWithMultipleLoggers() {
  try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
    context.register(ExampleThree.class, PlainLogger.class, JsonLogger.class);
    context.refresh();

    ExampleThree example = context.getBean(ExampleThree.class);
    example.runApps();
  }
}
{% endhighlight %}

When you run the above example, you get the output from both the LogService implementations.

    Data [some app data with ExampleThree] at 1545540204711
    {"log": { "message": "some app data with ExampleThree", "timestamp": 1545540204711 } }

Notice how the `stream()` method allows us to access all the available dependencies (0 or more).
You can even use the `orderedStream()` method to get the beans as defined by the order using `@Order` annotation on the beans.

API Doc:
<center><a href="https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/ObjectProvider.html" target="_blank"><img src="/assets/img/blogs/spring/stream.png"></a></center>

<br/>
Try removing all the LogService implementations and the code still works with no output as expected (see `testExampleThreeWithNoLoggers`).

So far, just using ObjectProvider<T>, we are able to handle all the cases where no dependency is available or there are one or more dependencies inside of the bean factory.

## More Use-cases

Lets dive into examples covering the 5.0 API.

### Adding a default implementation
Imagine if you have an application where you want to provide a default implementation programmatically when no dependencies are available for certain types.
How do you add a fallback mechanism?.

#### getIfAvailable/getIfUnique
I'll use the previous example where no dependencies of `LogService` exist but it should fallback to use the `PlainLogger` implementation.


API Doc:
<center><a href="https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/ObjectProvider.html" target="_blank">
<img src="/assets/img/blogs/spring/getifapi.png"></a></center>

<br/>

In the getLogService method of the following example, I use the `getIfAvailable` API that allows us to provide a `Supplier` if no candidates are available.

{% highlight java %}
public class ExampleFour {

  private ObjectProvider<LogService> logService;

  public ExampleFour(ObjectProvider<LogService> logService) {
    this.logService = logService;
  }

  LogService getLogService() {
    // use PlainLogger if not available.
    return logService.getIfAvailable(PlainLogger::new);
  }

  public void runApps() {
    getLogService().log("some app data with " + getClass().getSimpleName());
  }
}
{% endhighlight %}

Let's run with no dependencies that invokes the `getLogService()` on the ExampleFour bean.

{% highlight java %}
@Test
public void testExampleFourWithNoLoggers() {
  try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
    context.register(ExampleFour.class);
    context.refresh();

    ExampleFour example = context.getBean(ExampleFour.class);
    example.runApps();
  }
}
{% endhighlight %}

We get the output from the `PlainLogger` in this case.

    Data [some app data with ExampleFour] at 1545540354820

An astute reader might notice the issue here with using the `getIfAvailable` API.

What would happen when there are more than one implementations?

This would blow up (with NoUniqueBeanDefinitionException) since Spring cannot decide which one you need unless one of them is marked @Primary.

Rather we can use the `getIfUnique` API. It works for all the cases where there are no dependencies, one or more than one.
So when there are none or more than one dependencies, the fallback `PlainLogger` will be used. However if only one implementation of `LogService` is found, that would be used instead.

#### ifAvailable/ifUnique


API Doc:
<center><a href="https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/ObjectProvider.html" target="_blank"><img src="/assets/img/blogs/spring/ifAPI.png"></a></center>

<br/>
The following example uses the `ifAvailable` API that allows you to hook in a `Consumer` that accepts a bean instance if a dependency for the given type is found.

{% highlight java %}
public class ExampleFive {

  private final ObjectProvider<LogService> logService;

  public ExampleFive(ObjectProvider<LogService> logService) {
    this.logService = logService;
  }

  public void runApps() {
    logService.ifAvailable(e -> e.log("some app data with " + getClass().getSimpleName()));
  }
}

@Test
public void testExampleFiveWithOneLogger() {
  try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
    context.register(ExampleFive.class, JsonLogger.class);
    context.refresh();

    ExampleFive example = context.getBean(ExampleFive.class);
    example.runApps();
  }
}
{% endhighlight %}

When the `runApps` method is invoked, if a dependency is found, then the consumer is fired otherwise nothing happens. In the test above, you should see the output from the `JsonLogger`.

You can similarly use the `ifUnique` API that also takes in a `Consumer`. This comes in handy to support all cases as explained in the earlier scenario with `getIfUnique`.


## Summary
That wraps up the basic use-cases for ObjectProvider.
You can use it to retrieve a bean only if it actually exists or if a single candidate can be determined using programmatic resolution without blowing up at the wiring phase or while using the bean instances.
With Iterable and Stream support added in 5.1, we can easily support cases (with the same code) when zero or more dependencies exist for the bean of a given type.

The above code samples are available [here](https://github.com/rahulsh1/spring-objectprovider-examples)

## References <a name="references"></a>
[1] [Spring Framework 4.3 Core Updates](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3)

[2] [Spring Framework 5.1 Core Updates](https://spring.io/blog/2018/07/26/spring-framework-5-1-goes-rc1)

[3] [Spring Framework ObjectProvider API](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/ObjectProvider.html)
