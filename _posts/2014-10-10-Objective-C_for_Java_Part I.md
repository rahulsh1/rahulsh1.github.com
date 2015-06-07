---
layout: post
title: Objective-C for Java - Part I
category: tech
tags: Obj-C Objective-C to Java
year: 2014
month: 10
day: 10
published: true
summary: Objective-C ramp-up for Java programmers
---

Objective-C definitely has some learning curve even if you have prior experience with C and C++.
Coming from a Java background with more than few years experience, the curve was still rather steep. I had quite a hard time figuring out Java equivalents in Objective-C. The syntax itself takes a bit of time to get used to.

Also Objective-C has changed so much during the course of time that though there are plenty of learning material available, a lot of them carry out-dated content. So it becomes extra hard to figure the correct way to do things without risking using some obsolete stuff especially for someone new to Objective-C.

As I put myself through the transition and learnt things the hard way, hopefully these series of articles will make it easier and shorten the curve. Note it is very possible that contents of this article get outdated too(already if you work with `Swift`).
This article is meant for Java programmers who have just started learning Objective-C and are looking for some mapping guide. Note this is **not** a Objective-C tutorial and it would be good if you familiarize yourself with the concepts and syntax of Objective-C first.

This will be a multi-part article. In this first part, we shall cover the following:

**1. Using an Interface**

**2. Using Private Members and Methods &**

**3. Using Protected Methods** 


For all the above, we shall first see how this is done in Java and then look at its equivalent (or workaround/hack as some people might want to call it) part in Objective-C.

### 1. Inheritance

:point_right: **Java**

Lets first take an example of how we would write a class that implements a certain interface in Java.
The code should be self-explanatory.
{% highlight java linenos %} 
// Vehicle.java
public interface Vehicle {
  String SOME_CONSTANT = "config.txt";  // Implicitly public static and final

  void start();

  void stop();

  double getAverage();

}

// Car.java
public class Car implements Vehicle {

  @Override
  public void start() {
    System.out.println("Car started...");
  }

  @Override
  public void stop() {
    System.out.println("Car stopped...");
  }

  @Override
  public double getAverage() {
    return computeAverage();
  }

  public double computeAverage() {
    // do some calc
    return 25.00;
  }
}

// Main.java
public class Main {

  public static void main(String[] args) {
    Vehicle car = new Car();
    car.start();
    System.out.println("Average = " + car.getAverage());
    car.stop();
  }
}
{% endhighlight %}

:key: Output:
  
    Car started...
    Average = 25.0
    Car stopped...

:point_right: **Objective-C**

Lets see how the same thing can be acheived in Objective-C
If `Vehicle` is a strict interface that you want multiple classes to implement (which is true in this case), we can use the Objective-C Protocol which is equivalent to an Java Interface.
Otherwise you could simply create a Objective-C header which exposes the methods that your class implements that is also the interface exposed to the world.

The Interface(Protocol) would look like this:
{% highlight c linenos %} 
// Vehicle.h
#import <Foundation/Foundation.h>

@protocol Vehicle <NSObject>

// This will act just like any other property - but needs explicit synthesis
@property NSString* SOME_CONSTANT;

- (void) start;
- (void) stop;
- (double) getAverage;

@end

{% endhighlight %}

Lets implement the `Car` class now which "implements" the `Vehicle` Protocol.
Note the property `SOME_CONSTANT` needs to be explicitly synthesized otherwise you will get a compiler warning.

{% highlight c linenos %} 
// Car.h
#import <Foundation/Foundation.h>
@import "Vehicle.h"

@interface Car : NSObject <Vehicle>

- (double) computeAverage;

@end

// Car.m
#import "Car.h"
@implementation Car

@synthesize SOME_CONSTANT;

- (void)start {
  NSLog(@"Car started...");
}

- (void)stop {
  NSLog(@"Car stopped...");
}

- (double)getAverage {
  return [self computeAverage];
}

- (double)computeAverage {
  return 25.00;
}

@end

// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, char * argv[]) {
  @autoreleasepool {
    id <Vehicle> car = [[Car alloc] init]; // Note id is an implicit pointer type

    [car start];
    NSLog(@"Average = %f", [car getAverage]);
    [car stop];

    [car setSOME_CONSTANT:@"config.txt"];
    NSLog(@"Constant=%@", [car SOME_CONSTANT]);

    // Calling public method from Car class
    if ([car conformsToProtocol:@protocol(Vehicle)]) {
      [(Car *)car computeAverage];
    }
  }
  return 0;
}

{% endhighlight %}

Note how we defined an object of type Vehicle as `id` and how we would need to typecast in order to call methods on the `Car` object.

### 2. Private Members/Method
Java simply allows one to use the **private** keyword and our job is done.

:point_right: **Java**

{% highlight java linenos %} 
public class Car implements Vehicle {
  private String engineType;
  ...

  private int getNumberOfWheels() {
    return 4;
  }
  private boolean isEconomical() {
    return computeAverage() > 25;
  }
}
{% endhighlight %}

:point_right: **Objective-C**

Private Members can be easily defined at the start of the implementation definition as can be seen below.
This property is now accessible only within this class. Its a good practice to put an underscore for private variables. This property can be directly accessed in the following way:

{% highlight c linenos %} 
@implementation Car {
 // Private members are defined here
 NSString *_engineType;
}
...
self->_engineType = @"TurboV8";
NSString *data = self->_engineType;
...
{% endhighlight %}

In case of methods, since there is no direct way of specifying private methods, one way would be to define the method only in the implementation(.m) file so it remains hidden. By not defining it in the .h file, its kind of hidden from the outside world.

This approach may work if you have very few private methods that you can track and make sure that these are implemented.
If you have several private methods, its a good practice to use **Extensions** instead.

Extensions let you declare a formal API that is also validated by the compiler (you'll see a warning) if you dont implement it. Lets see the Objective-C implementation.

{% highlight c linenos %} 
// Car.m
#import "Car.h"

// The class extension
@interface Car ()
- (int)getNumberOfWheels;
- (BOOL)isEconomical;
@end

@implementation Car {
  // Private members are defined here
  NSString *_engineType;
}
  ...

- (int) getNumberOfWheels {
  return 4;
}
- (BOOL) isEconomical {
    return [self computeAverage] > 25 ? YES: NO;
}
@end

{% endhighlight %}

###3. Protected Methods

:point_right: **Java**

Lets say we have a method `adjustSpeed` that should be implemented by only sub-classes of `Car`.

{% highlight java linenos %} 
public class Car implements Vehicle {
  ..

  protected void adjustSpeed() {
    System.out.println("To be implemented by sub-classes");
  }
}  

public class AudiV6 extends Car {

  protected void adjustSpeed() {
    System.out.println("Adjusting Audi's speed !!!");
  }

  public void start() {
    System.out.println("Vhoooooooooooooooo");
  }
}

// Main method
public static void main(String[] args) {
  Car car = new AudiV6();
  car.start();
  // wont work here
  //car.adjustSpeed()
  car.stop();
}

{% endhighlight %}

:point_right: **Objective-C**

Since there is no protected keyword, we can use `Categories` to implement this behavior.

{% highlight c linenos %} 
// Car+Inherited.h
#import "Car.h"

@interface Car (Inherited)

- (void)adjustSpeed;

@end

// Car+Inherited.m
@implementation Car (Inherited)

- (void)adjustSpeed {
  NSLog(@"To be implemented by sub-classes");
}

@end

// AudiV6.h

#import "Car.h"

@interface AudiV6 : Car 

@end

// AudiV6.m
#import "AudiV6.h"
#import "Car+Inherited.h"
@implementation AudiV6

- (void) start {
  NSLog(@"Vhoooooooooooooooo");
}

- (void) adjustSpeed {
  NSLog(@"Adjusting Audi's speed !!!");
}

@end

// main.m
#import <Foundation/Foundation.h>
#import "Car.h"
#import "AudiV6.h"

int main(int argc, const char * argv[]) {
  Car *car1 = [[AudiV6 alloc] init];
  [car1 start];

  // This wont work since the adjustSpeed is not part of Car.h
  //[car1 adjustSpeed];
  [car stop];

}
{% endhighlight %}

Make sure you keep the `Car+Inherited.h` file within the project and not expose it outside your project(do not include in any public header file).

If the main program knows about it and includes the `Car+Inherited.h`, it can invoke the `adjustSpeed` method since the contents of the category become part of Car implementation.

That wraps the first part of this series :running:, the next one will cover further topics on Objective-C equivalents from Java. Till then... :v:


