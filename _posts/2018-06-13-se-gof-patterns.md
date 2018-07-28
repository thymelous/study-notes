---
layout: post
title: "Software Engineering: GoF Patterns (Incomplete)"
date: 2018-06-13
---
_Gang of Four_'s design patterns solve common object-oriented software design problems.
While it is undoubtedly useful to be at least familiar with them, it is important
to stress how important it is to not overuse them, thus overcomplicating the code.

The patterns are split into _creational_, concerned with how objects are created;
_structural_, defining relationships between classes,
and _behavioral_, establishing communication between classes.

## Creational patterns

**Singleton** ensures that only a single instance of a class is created, which is required
when there's a single underlying component (e.g. a system device), access to which needs to
be controlled (queueing, ...) by an arbiter.

```java
public class Singleton {
    private static Singleton singletonInst = null;

    private Singleton() { }

    public static synchronized Singleton getInstance() {
        if (singletonInst == null) singletonInst = new Singleton();
        return singletonInst;
    } 
}
```

**Factory method** can be used instead of a constructor to create an object without specifying
the exact class of it. The textbook definiton is a little bit different from how the word is
usually used: it involves creating two interfaces, a `Product` that is created and a `Creator`
that constructs it. The latter features a `factoryMethod` that produces the former, 

<img src="https://upload.wikimedia.org/wikipedia/commons/4/43/W3sDesign_Factory_Method_Design_Pattern_UML.jpg" />

...

## Structural patterns

**Adapter** implements an interface required by the client on top of
an existing class (adaptee) with a different, incompatible interface.

<img src="https://upload.wikimedia.org/wikipedia/commons/d/d7/ObjectAdapter.png" />

**Bridge** is used when abstraction and implementation should be defined
and extended independently from each other.

<img width="450" src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/cf/Bridge_UML_class_diagram.svg/1000px-Bridge_UML_class_diagram.svg.png" />

**Composite** defines a unified `Component` interface for objects that
may (`Composite`) or may not (`Leaf`) have other objects (`Component`s) as children,
thus creating a tree hierarchy. `Composite` nodes have operations to manipulate children
and propagate method invocations (`operation`) to them.

<img width="450" src="https://upload.wikimedia.org/wikipedia/commons/5/5a/Composite_UML_class_diagram_%28fixed%29.svg" />

```java
interface Component { public void operation(); }

class Composite implements Component {
    private List<Component> children = new ArrayList<>();

    public void operation() {
        for (Component c : children) c.operation();
    }

    public void add(Component c) { children.add(c); }

    public void remove(Component c) { children.remove(c); }
}

class Leaf implements Component { public void operation() { } }
```

**Decorator** extends the functionality of objects by wrapping them into one or more decorator classes. Decorators
do not affect other objects of the same class.

<img width="450" src="https://upload.wikimedia.org/wikipedia/commons/e/e9/Decorator_UML_class_diagram.svg" />

**Facade** provides a simple interface to a complex or poorly-designed subsystem. It provides additional benefits
for libraries, reducing outside dependencies on the inner workings and providing convenience methods for common tasks.

```java
class A { public void doSomething() { } }

class B { public void doSomethingElseEntirely() { } }

class C { public void needsToBeRunFirst() { } }

class Facade {
  private A a; private B b; private C c;

  public Facade() { a = ...; b = ...; c = ...; }

  public void performAction() {
    c.needsToBeRunFirst();
    a.doSomething();
    b.doSomethingElseEntirely();
  }
}
```

## Behavioral patterns

**Iterator** allows the client to sequentially access elements of an aggregate object without knowing its structure.
Iterator encapsulates accessing and traversing the agregate, which allows new traversal operations to be defined
independently from the aggregate interface.

<img width="450" src="https://upload.wikimedia.org/wikipedia/commons/1/13/Iterator_UML_class_diagram.svg" />

**Mediator** is used to encapsulate and centralize communication between classes. Instead of communicating with each other
directly, objects send messages through the mediator, reducing coupling.

**Memento** encapsulates the state of an object at a certain point in time and allows the object to be restored to this state.

_...to be continued_
