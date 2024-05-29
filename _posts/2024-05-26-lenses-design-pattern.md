---
title: 'Lenses design pattern'

subtitle: "playing with a composition of functions"
author:
  - rczmdp
tags: [ design patterns, immutability, lenses ]
description: |
  `Lenses` pattern provide a way of scalable updating immutable data structures.  
  Article is inspired by the book "Functional And Reactive Domain Modeling" by Debasish Ghosh. 
  It provides a sample implementation of `lenses` pattern and examples of it's usage.
---

This article is inspired by the
book [Functional And Reactive Domain Modeling](https://www.manning.com/books/functional-and-reactive-domain-modeling) by
Debasish Ghosh. In the book there are examples in scala. This article extends that part of the book with sample
implementation of `lenses` pattern and examples of its usage and java.

`Lenses` pattern provide a way of scalable updating immutable data structures. Scalable in this case stands for
organizing the code, when the complexity of the data is growing. Updating immutable seems like impossible. How it's
possible to change the object that is unchangeable?

### Mutable example

Let's look at the real-life example: your car tire is pinched. As a consequence you go to a local mechanic.  
What will happen, mechanic will get a pinched tire, take it off, fix it, put it back. That seems fine solution.
Motorization business is operating like this for more than a hundred years.
This is how mutable data structure works:

```java 
Car repairPinchedTire(Car brokenCar) {
  return brokenCar.getFrontLeftTire().setPreasure(2.2f);
}
```

### Immutable example

On the other hand, our company is having a fleet of thousands cars. Let's model the situation from the perspective of
the driver, when car he is using is pinched.
So the driver is not assigned to a car, driver just need a car. So from a driver perspective the car goes into mechanic,
and the car is returned from mechanic.
So in his case situation might look like this:

```java
Car repairPinchedTire(Car brokenCar) {
  var brokenTire = brokenCar.getFrontLeftTire();
  var fixedTire = brokenTire.withPreassure(2.2f);
  return brokenCar.withFrontLeftTire(fixedTire);
}
```

What is the difference? The difference is that in a second case, driver consider gets a different car - a different
object, but with the same properties. The previous car still might exist.
How that is possible? The old car can exist in the memory of the driver: as long as it won't be garbage collected. It
also might exist in a mechanic registry as a history record.

### Complexity is growing

In real life car is not just a four wheels and engine. The wheels itself contains a few elements, each is coming for
different producers, each having its own model, series, size, durability.
The same apply to the engine: spark plugs, piston and many more things. Each of those is an object on its own rights:
have a manufacturer, model, series, and its own functions.

Writing a repairedPinchedTire might not be as straight forward, when there is a deep level of nesting and here comes
design pattern: `lenses`. The presented code won't make things looks as easy as the first mutable example, there will be
more code, but the schema will be prepared to handle such parts replacement when complexity of the object is growing.

Let's say that our mechanic workshop get a broken car.
Initially we need to figure out what elements are not working correctly.
After making a basic test, it's obvious that there is something wrong with the engine.
So we can narrow our focus to the engine. The manual for the car contains instructions how to get engine out of chasis
and how to put it back.
The engine goes to the engine specialist. After checkup, it was discovered issues with a spark plug and a piston.
In engine manual there are instruction how to take out and put in a spark plugs and a piston.

So the full process will look like:

* get the engine out of car
* get the spark plug out of engine
* put new spark plug into the engine
* get piston out of engine
* regenerate the piston
* put a new pisto to the engine
* put engine back to car

### Data model

Before going directly to job, let's first take a detailed look at clases that are related to car:

```java

@With
@Value
@Builder
class Car {

  String id;
  Engine engine;

  @UtilityClass
  public static class Lenses {
    static final Lens<Car, String> id = Lens.of(Car::getId, Car::withId);
    static final Lens<Car, Engine> engine = Lens.of(Car::getEngine, Car::withEngine);
  }
}

@With
@Value
@Builder
class Engine {

  Piston piston;
  SparkPlug sparkPlug;
  float power;

  @UtilityClass
  public static class Lenses {
    static final Lens<Engine, Piston> piston = Lens.of(Engine::getPiston, Engine::withPiston);
    static final Lens<Engine, SparkPlug> sparkPlug = Lens.of(Engine::getSparkPlug, Engine::withSparkPlug);
    static final Lens<Engine, Float> power = Lens.of(Engine::getPower, Engine::withPower);
  }
}

@With
@Value
@Builder
class SparkPlug {

  String producer;
  String shortCode;
  SparkPlugState state;

  enum SparkPlugState {
    NORMAL, ELECTRODE_MELTED, OILEDUP, SOOT_DEPOSIT
  }

  @UtilityClass
  public static class Lenses {
    static final Lens<SparkPlug, String> producer = Lens.of(SparkPlug::getProducer, SparkPlug::withProducer);
    static final Lens<SparkPlug, String> shortCode = Lens.of(SparkPlug::getShortCode, SparkPlug::withShortCode);
    static final Lens<SparkPlug, SparkPlugState> state = Lens.of(SparkPlug::getState, SparkPlug::withState);
  }
}
```

Step by step:

* `@With @Value @Builder` are lombok annotations, meaning that all properties of the object will be final -
  immutable. The all args constructor and the builder will be generated. In addition, for each property there will be
  defined a method, that returns a new object of the given class, but with a replaced property. Check lombok project for
  more details.
* `@UtilityClass` - make class a final, with no public constructors - additional useful thing from lombok project.
* inner classes `Car.Lenses`, `Engine.Lenses`, `SparkPlug.Lenses` , `Piston.Lenses` - provides a pair of functions:
  allowing to get and replace a given property.

### Usage examples

For now let's assume just the engine is broken, and we are missing competences to fix the engine.

```java
Car repairedCar = Lens.focus(Car.Lenses.engine)
  .rebuildWith(EngineProvider.get(brokenCar))
  .apply(brokenCar);
```

Let's analyze it deeper:
```Lens.focuss(Car.Lenses.engine)``` create a 'lens', that allow for a given car focus and replace an engine.
``rebuildWith(EngineProvider.get(brokenCar))`` returns a function, accepting a Car, and returning a Car, it will be
constructed with a new engine from `EngineProvider`.
Those lines are creating a recipe, for replacing the broken engine. The replacement happen when we are applying this
function to a broken car.
The behavior is exactly like below, except that we are focused on a function composition, that later can be applied.

```java
Car repariedCar = brokenCar.withEngine(EngineProvider.get(brokenCar));
```

At this point, using lenses is for sure overkill. After some time we might get competences to repair engines.

```java
Car repairedCar = Lens.focus(Car.Lenses.engine)
  .focus(Engine.Lenses.piston)
  .focus(Piston.Lenses.operational)
  .rebuildWith(true)
  .apply(brokenCar);
```

At this level function composition is getting more complicated. Let's combine it with spark plug replacement.

```java
Car repairedCar = Lens.focus(Car.Lenses.engine)
  .split(Lens.focus(Engine.Lenses.sparkPlug)
    .focus(SparkPlug.Lenses.state)
    .rebuildWith(SparkPlug.SparkPlugState.NORMAL))
  .focus(Engine.Lenses.piston)
  .focus(Piston.Lenses.operational)
  .rebuildWith(true)
  .apply(brokenCar);
``` 

Benefits are becoming visible: with the one go I'm replacing two elements of the engine. Having those lenses might be a
helpful, when having more and more complex classes.

### Conclusions

Would I use this code in production? No. Let me explain:

* requires external library or implementing the monad, similar
  to [this](https://github.com/rczyzewski/tortilla/blob/main/tortilla/src/main/java/io/github/rczyzewski/tortilla/lens/Lens.java).
  Importing this will cause extra code, not related to business logic.
* the library providing lenses is my own, not well recognized, not used by anyone, still experimental
* the above code using lenses is actually more verbose
* require creating a utility classes, like: `Car.Lenses`, `Engine.Lenses`, `SparkPlug.Lenses` , `Piston.Lenses`
* pattern is not well known - though is harder to adopt in a team of developers

Are there any advantages of this pattern? 
* yes, but require helper object to make it shine
* promotes composition of functions, that change the way of thinking about the code


Why to bother with yourself with these pieces of code? Because it's the exercise for composition of functions. It's a
kind of food for a brain. It might happen that sooner or later someone will stumble upon a problem where such a
construct might be helpful - so you will be already prepared. The code samples are available
with [tortilla](https://github.com/rczyzewski/tortilla) library.





