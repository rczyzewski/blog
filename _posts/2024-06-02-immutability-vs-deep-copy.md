---
title: 'Immutable objects vs deep copy'
subtitle: "How to make an object immutable and the relationship between immutability and shallow/deep cloning"
author:
  - rczmdp
categories: [ java, immutability, deep copy, shallow copy ]
tags: [ java ]
description: |
  Describes the difference between shallow/deep copy of an object. Then it explains the immutability concept based on a 
  `String` class, finally presents an immutable alternative to previous examples.
---

This blog is gaining readers: my friend has reviewed the article about
the [lenses](https://rczyzewski.github.io/blog/posts/lenses-design-pattern/), which describes the construction of an
immutable object, composed of possibly many layers of immutable objects. He was curious about the relationship
immutability concept with a deep copy.

This article focuses definition of immutability, presents the implementation of shallow copy/deep copy and the immutable
object, and explains the differences between them.

In the first part let's focus on a shallow copy. I will skip an interface `Cloneable` and Lombok annotations will be
used in the example. In the next part, I will show the difference between a shallow copy and a deep copy. Then I will
explain the behavior of an immutable object: String. At the end, I will rewrite the example in an immutable manner.

The examples below are written in Java. The full code is available as
a [gist](https://gist.github.com/rczyzewski/14cfbe4c676d29297cf5f863c27e7112). There is one common interface used for
all cases:

```java

  interface Person {
    String getName();

    int getHeight();

    List<String> getHobbies();

    static void compareTwoPersons(Logger log, Person p1, Person p2) {
      log.info("p1: {}", p1);
      log.info("p1: {}", p2);
      log.info("are persons same object: {}, are equal: {}", p1 == p2, p1.equals(p2));
      log.info("the name is the same: {}, are equal: {}", p1.getName() == p2.getName(), p1.getName().equals(p2.getName()));
      log.info("the height is the same: {}", p1.getHeight() == p2.getHeight());
      log.info("the hobbies is the same: {}, are equal: {}", p1.getHeight() == p2.getHeight(), p1.getHobbies().equals(p2.getHobbies()));
    }
  }
```

The purpose of it is to be able to compare the objects when they are changing.

## Shallow copy

Let's first focus on shallow copy. For this exercise, let's consider an object as a piece of memory. In that piece of
memory, there are primitive values and/or references. For example object of a class presented below contains a reference
to a String object for the name, the primitive value for height, and the reference to a List object for keeping track of
hobbies.

```java

@Data
@AllArgsConstructor
public static class ShallowPerson implements Person {
  String name;
  int height;
  List<String> hobbies;

  ShallowPerson(Person person) {
    this(person.getName(), person.getHeight(), person.getHobbies());
  }
}
```

Let's use [lombok](https://projectlombok.org/features/) documentation to explain annotations:

* `@AllArgsConstructor` generates a constructor with 1 parameter for each field in your class. Fields marked
  with `@NonNull` results in null checks on those parameters.

* `@Data` generates all the boilerplate that is normally associated with POJOs getters for all fields, setters for all
  non-final fields, and appropriate toString, equals, and hashCode implementations that involve the fields of the class,
  and a constructor that initializes all final fields, as well as all non-final fields with no initializer that have
  been marked with `@NonNull`, to ensure the field is never null.

Shallow copy will just create a new object, with the same values. Let's take a look at copying
constructor `Person(Person person)`.

Let's consider the relations between two object persons, one constructed with a copying constructor.

```java

  @Test
  void shallowCopy1() {
    ArrayList<String> hobbies = new ArrayList<String>();
    Person p1 = new Person("Hannah", 153, hobbies);
    Person p2 = new Person(p1);

    log.info("-------------- initial");
    compareTwoPersons(p1, p2);

    log.info("-------------- name changed");
    p2.setName("Montana");
    compareTwoPersons(p1, p2);

    log.info("-------------- height changed");
    p2.setHeight(154);
    compareTwoPersons(p1, p2);

    log.info("-------------- hobbies changed");
    p1.getHobbies().add("crime books");
    compareTwoPersons(p1, p2);


  }

  void compareTwoPersons(Person p1, Person p2) {
    log.info("p1: {}", p1);
    log.info("p1: {}", p2);
    log.info("are persons same object: {}, are equal: {}", p1 == p2, p1.equals(p2));
    log.info("the name is the same: {}, are equal: {}", p1.getName() == p2.getName(), p1.getName().equals(p2.getName()));
    log.info("the height is the same: {}", p1.getHeight() == p2.getHeight());
    log.info("the hobbies is the same: {}, are equal: {}", p1.getHobbies() == p2.getHobbies(), p1.getHobbies().equals(p2.getHobbies()));
  }
```

Changing the name in object `p2` is changing the value of the reference: it is assigning a different reference. The
original object of String is not changed. Object `p1` is still using the original name, but object `p2` is now having a
reference to a different object. Changing a value for `height` affects only an object `p2`, the primitive values are
stored within the object itself, so after applying a copying constructor, each object has its area of memory where the
value is stored. There is something a little bit different happening when the hobbies are changed. In this case, we are
not changing the object `p2` but the `List<String>`, that both objects `p1` and `p2` are referencing.

## Deep copy

The deep copy concept is very similar to a shallow copy: with one key difference. All mutable objects will be copied
recursively too. Changing the example from the previous part, the main difference will be about a mutable property:
hobbies. In our case, this mutable element is the List of hobbies.

```java
  @Data
  @AllArgsConstructor
  public static class DeepPerson implements Person {
    String name;
    int height;
    List<String> hobbies;

    DeepPerson(Person person) {
      this(person.getName(), person.getHeight(), new ArrayList<>(person.getHobbies()));
    }
  }
```

The difference is that when we are changing the object of hobbies: when we are deep copying we are creating a new
instance of the list, and that new instance is assigned to a new object. That is opposed to shallow copy, where we just
copied the references, so changing the object behind the reference was changing the value in both of them.

## Immutability

According to [Cambridge Dictionary](https://dictionary.cambridge.org/dictionary/english/immutability) the word
immutability stands for: `being unable to be changed`. The very standard example of an immutable object is `String`: we
already have seen its behavior in both shallow and deep copy examples. This time let's take a look String object, and
how it is assigned to a variable.

```java
  String str = "String";
  String strOriginal = str;
  str.toUpperCase();
  assert str == "String";
  str = "AnotherString";
  assert str = "AnotherString";
  assert strOriginal = "String";
```

Let's analyze the above code line by line:

* line 1 - assigning to a variable `str` a reference to an object `String`;
* line 2 - assigning to a variable `strReference` a reference to the same object as variable `str`
* line 3 - executing a method on an object behind a variable `str`, method is returning a reference to a new object
* line 4 - checking if behind 'str' there is still "String", so line 3 has no effect
* line 5 - assigning to a variable `str` an object "AnotherString"
* line 6 - checking if the statement in line 4 has changed the assignment
* line 7 - check if a change in line 5, hasn't affected an object behind variable `strOriginal`

So far, so good, one object is immutable: after it's created, it can't be changed, but new objects can be obtained, that
are modifications to the original one: for example with a method `toUpperCase`.

Let's now make our class implement a Person, behaving like that.

```java

  @Value
  @With
  class ImmutablePerson implements Person {
    String name;
    int height;
    List<String> hobbies;
  }
```

These two were not used before annotations from  [lombok](https://projectlombok.org/features/):

* `@Value` is taking care of making attributes private and final, the whole class is final: it can't have subclasses. It
  is similar to @Data: it also generates equals/hashCode/toString and getters, and it also enriches a class
  with `AllArgConstructor`.
* The next best alternative to a setter for an immutable property is to construct a clone of the object but with a new
  value for this one field. A method to generate this clone is precisely what @With generates: a withFieldName(newValue)
  method which produces a clone except for the new value for the associated field.

Let's take a closer look at method `withName`: the returned object is ImmutablePerson. It's important that the returned
object is not the same object, where the method has been invoked. It's a different object, containing two references.
The 'name' is referencing a different string, than in the original object. The references to the second attribute is
referring the same object, as the original immutable person.

```java

  ImmutablePerson p1 = new ImmutablePerson("Hannah", 153, List.of());
  ImmutablePerson p2 = p1;

  log.info("-------------- initial");
  Person.compareTwoPersons(log, p1, p2);

  log.info("-------------- name changed");
  Person.compareTwoPersons(log, p1, p2.withName("Montana"));

  log.info("-------------- height changed");
  Person.compareTwoPersons(log, p1, p2.withHeight(177));

  log.info("-------------- hobbies changed");
  Person.compareTwoPersons(log, p1, p2.withHobbies(List.of("crime books")));
```

So if the object can't be changed, there is no need to copy it. But if we want to have a different version of an object,
we can reuse all the references from the original object and create a new object, referencing the same properties,
except those that we wanted to be changed. In a sense, we might think about it as a shallow copy + modification,
although all fields are final and objects inside are immutable.

Then why write so much, when we can get the same results by copying objects and using the setter? Why the `immutability`
concept is so important? If such objects are shared between threads, those threads can't make modifications to them, so
we are avoiding the need to synchronize them.

Please hit the like button and subscribe to the channel. 
