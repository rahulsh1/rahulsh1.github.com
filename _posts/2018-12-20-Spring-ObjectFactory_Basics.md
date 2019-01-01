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

When Spring Framework 4.3 was released, it introduced `ObjectProvider`. As Spring's blog [1] mentions, this provided an extension to the existing `ObjectFactory` interface with handy signatures such as getIfAvailable, getIfUnique etc to retrieve a bean only if it actually exists or if a single candidate can be determined (such as a primary candidate in case of multiple matching beans). This also improved the programmatic resolution of dependencies.

With 5.0 release, this was improved to support newer APIs that allowed you to pass a `Supplier` for the getIfAvailable/getIfUnique APIs and a `Consumer` for the ifAvailable/ifUnique APIs.

And with 5.1 release, this was extended further to give it `Iterable` and `Stream` support [2].


## Diving into ObjectProvider
Let's take a look at the things you can do with ObjectProvider. We'll dive into the simple usecases first and then cover the newer APIs too.

The code discussed below is available [here](https://github.com/rahulsh1/spring-objectprovider-examples).

### No dependencies
Let's say I've this simple Interface for a logging service as given below:

    public interface LogService {
        void log(String data);
    }

I am going to use this LogService inside the Spring Bean as shown.

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

The above Spring bean wires in the LogService dependency using constructor-based injection.

You don't really need to mention `@Autowired` as part of the Constructor param as this is the only Constructor. This support came in with Spring 4.3 that allows `Implicit constructor injection for single-constructor` scenarios. I also don't need the `@Component` since I am explicitly registering it with the `ApplicationContext`. I am going to skip both in the later examples.

I'll now wire this up in the following test case and we shall see what happens when there are no dependencies for `LogService`.

    @Test(expected = UnsatisfiedDependencyException.class)
    public void testExampleOneMissingBean() {
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
            context.register(ExampleOne.class);
            context.refresh();
            context.start();

            ExampleOne example = context.getBean(ExampleOne.class);
            example.runApps();
        }
    }

Notice that no dependencies of `LogService` were registered with the ApplicationContext. There is only one User defined bean.
As expected, we get the `NoSuchBeanDefinitionException` for the field logService in `ExampleOne`.

> Throws Unsatisfied dependency expressed through field 'logService'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.example.beans.LogService' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

You may need to run your application in certain environments where it is normal for some dependencies to be missing (i.e. they are optional) and you don't want your application to blow up like above.

How do you go above handling such a case then?

### Making it Optional
Let’s see how the `ObjectProvider` can come to our rescue here to handle optional dependencies.

I'll update the type from `LogService` to `ObjectProvider<LogService>` and also modify the runApps method to use the `ifAvailable` API.

    public class ExampleTwo {

        private ObjectProvider<LogService> logService;

        public ExampleTwo(ObjectProvider<LogService> logService) {
            this.logService = logService;
        }

        public void runApps() {
            logService.ifAvailable(e -> e.log("some data"));
        }
    }

Now when I run the same test as before, we don't get any exception and things are all good. No output is printed as expected.

Let's make sure this works even when a dependency actually exists. To do that, we register the `PlainLogger` bean as well with the ApplicationContext as part of `context.register(...)`.

    @Test
    public void testExampleTwoWithOneLogger() {
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
            context.register(ExampleTwo.class, PlainLogger.class);  // Register the dependency
            context.refresh();
            context.start();

            ExampleTwo example = context.getBean(ExampleTwo.class);
            example.runApps();
        }
    }

This time I get some output as expected:

    Data [some data] at 1545539958913

> Note:
You can also use the `java.util.Optional<T>` instead of `ObjectProvider<T>` but it works well only when zero/one implementation(s) exists.

### More than one dependency
Now what happens if there are more than one dependencies of `LogService`, which one would be used with the above? Any guesses?

I am registering two implementations here for `LogService`, namely `PlainLogger` and `JsonLogger`.

    @Test
    public void testExampleTwoWithMultipleLoggers() {
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
            context.register(ExampleTwo.class, PlainLogger.class, JsonLogger.class);
            context.refresh();
            context.start();

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
        ...
      }
    }

This would not work. Notice where this fails though. The wiring is fine though however when calling `runApps` Spring doesn't know which one to choose for you.

Since Spring cannot decide which specific implementation you want (and neither bean is marked as `@Primary` too), this would throw an expection.
> org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.example.service.LogService' available: expected single matching bean but found 2: plainLogger,jsonLogger

One fix for above would be to mark of the implementations as `@Primary`. But what if you needed both the implementations?

To do that, I'll now use the API introduced in 5.1 for this (this is not supported pre-5.1 though).

We don't need to make changes to the type. The type remains the same, it is still `ObjectProvider<LogService>` and not `ObjectProvider<List<LogService>>`.

However the API to access the beans is a bit different. We use the `stream` API to access the dependencies. We could have used an enhanced for loop as well since the `ObjectProvider` interface extends `Iterable`. See the updated implementation of `runApps`

    public class ExampleThree {

        private ObjectProvider<LogService> logService;

        public ExampleThree(ObjectProvider<LogService> logService) {
            this.logService = logService;
        }

        public void runApps() {
            logService.stream().forEach(e -> e.log("some app data with " + getClass().getSimpleName()));
        }
    }

Let's test this out.

    public void testExampleThreeWithMultipleLoggers() {
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
            context.register(ExampleThree.class, PlainLogger.class, JsonLogger.class);
            context.refresh();
            context.start();

            ExampleThree example = context.getBean(ExampleThree.class);
            example.runApps();
        }
    }

When you run the above example, you get the output from both the LogService implementations.

    Data [some app data with ExampleThree] at 1545540204711
    {"log": { "message": "some app data with ExampleThree", "timestamp": 1545540204711 } }

Notice how the `stream` API allows us to access all the available dependencies (0 or more).

Try removing all the LogService implementations and the code still works with no output as expected (see `testExampleThreeWithNoLoggers`).

So far, just using ObjectProvider<T>, we are able to handle all the cases where no dependency is available or there are one or more dependencies inside of the bean factory.

## More Use-cases

Lets dive into examples covering the 5.0 API.

### Adding a default implementation
Imagine if you have an application where you want to provide a default implementation programmatically when no dependencies are available for certain types.
How do you add a fallback mechanism?.

I'll use the previous example where no dependencies of `LogService` exist but I want to fallback to use the `PlainLogger` implementation.
So in the getLogService method, I use the `getIfUnique` API that allows us to provide a `Supplier` if no unique candidates are available.

    public class ExampleFour {

        private ObjectProvider<LogService> logService;

        public ExampleFour(ObjectProvider<LogService> logService) {
            this.logService = logService;
        }

        LogService getLogService() {
            // use PlainLogger if not unique.
            return logService.getIfUnique(PlainLogger::new);
        }

        public void runApps() {
            logService.stream().forEach(e -> e.log("some app data with " + getClass().getSimpleName()));
        }
    }

Let's run with no dependencies.

    public void testExampleFourWithNoLoggers() {
        try (AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext()) {
            context.register(ExampleFour.class);
            context.refresh();
            context.start();

            ExampleFour example = context.getBean(ExampleFour.class);
            example.getLogService().log("Hello Logger");
        }
    }

We get the output from the `PlainLogger` in this case.

    Data [Hello Logger] at 1545540354820

Note that I've used the `getIfUnique` API and not the `getIfAvailable` API.

However you may also use the `getIfAvailable` API, but that would blow up (with NoUniqueBeanDefinitionException) when more than one dependencies are found.

With `getIfUnique`, it works for both the cases where there are no dependencies or more than one. In both cases, the `PlainLogger` will be used. If only one implementation of `LogService` is found, that would be used instead.


The following example uses the `ifAvailable` API that allows us to hook in a `Consumer` that takes a bean instance if one if found.

    public class ExampleFive {

      private final ObjectProvider<LogService> logService;

      public ExampleFive(ObjectProvider<LogService> logService) {
        this.logService = logService;
      }

      public void runApps() {
        logService.ifAvailable(e -> e.log("some app data with " + getClass().getSimpleName()));
      }

      public int numDependencies() {
        int[] count = {0};
        logService.stream().forEach(e -> count[0]++);
        return count[0];
      }
    }

When the `runApps` method is invoked, if a dependency is found, then the consumer is fired otherwise nothing happens. You can also use the `ifUnique` API that also takes in a `Consumer`.

## Summary
That wraps up the basic use-cases for ObjectProvider.
You can use it to retrieve a bean only if it actually exists or if a single candidate can be determined using programmatic resolution without blowing up at the wiring phase or while using the bean instances.
With Iterable and Stream support added in 5.1, we can easily support cases (with the same code) when zero or more dependencies exist for the bean of a given type.

The above example code is available [here](https://github.com/rahulsh1/spring-objectprovider-examples)

## References
[1] [Spring Framework 4.3 Core Updates](https://spring.io/blog/2016/03/04/core-container-refinements-in-spring-framework-4-3)

[2] [Spring Framework 5.1 Core Updates](https://spring.io/blog/2018/07/26/spring-framework-5-1-goes-rc1)

[3] [Spring Framework ObjectProvider API](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/beans/factory/ObjectProvider.html)
