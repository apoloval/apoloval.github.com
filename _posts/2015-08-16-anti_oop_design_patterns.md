---
title:  "Anti-OOP Design Patterns"
date:   2015-08-17
categories: oop
---

Any good programmer is familiarized with design patterns. Even not-so-good ones
ever heard about singleton, strategy, decorator, observer, etc. What is not so
common is to find programmers who are aware about the reasons that lead them
to use such patterns. For most of us patterns are just a tool. Something that
works. And we don't even try to reason about where the problem they solve comes
from. We apply them and continue coding.

But, if you dare to break the confirmism and reason about our good friends the
design patterns, you might realise **they are there to fight a surprising enemy:
the object-oriented programming**. Don't believe me? Let's analize some popular
design patterns from this perspective.



## Singleton Pattern

Simple but elegant. We use [Singleton pattern][1] to define a class that only
can have one instance. In Java:

{% highlight java %}
class Application extends Configurable {

  private static Application instance;

  private Application() { ... }

  public static Application getInstance() {
    if (instance == null)
      instance = new Application();
    return instance;
  }

  // More methods here
  // ...
}
{% endhighlight %}

This code is self-descriptive. By declaring a private method we cannot create
new instances outside this class. The `getInstance()` method uses that private
constructor to return the single instance of the class in the whole system.

Right. What's going on here? What is exactly Singleton solving? In most popular
OOP languages, a class is a factory of objects. As such, you can use it to
create as many objects as you want (or fit into memory). But the real-world
things represented by such instances may be unique by nature. E.g., the class
representing your application in a GUI library. Having more than one instance of
`Application` would mean your program represents more than one application. And
that's a restriction of the operating system (one process, one app).

It is very clear that singleton pattern is fighting against the OOP rules. You
need a single object in your system, but the language rules prevent that. You
are using singleton pattern to cheat the OOP. To break its rules. Singleton is
a clear example of a anti-OOP design pattern.

Is there a way to use OOP without cheating with singletons? Of course it is!
It's just a matter of having a language that accepts classes that can have only
one instance. Let's see an example in Scala.

{% highlight scala %}
object Application extends Configurable {
  // More methods here
  // ...
}
{% endhighlight %}

This is really simple and really elegant. In Scala, there is a special case of a
class that only have one instance. It is declared with the keyword `object`
instead of `class`. In this example, the identifier `Application` may refer to
both the class and the instance. Thus the class is not an object factory any
longer. We got our single instance type without cheating the language.

## Observer Pattern

Another really popular pattern. [Observer pattern][2] is used to notify state
changes on an object to other objects. Some example:

{% highlight java %}
interface OnMouseClick {
  void onClick(Control control);
}

class Button extends Control {
  public void addOnClick(OnMouseClick onClick) { /* ... */ }

  // More code here
  // ...
}
{% endhighlight %}

Let's say we have a GUI library with a `Button` class that represents a button
UI control. The `OnMouseClick` interface represents an action that would be
executed when a mouse click event is detected on a given control. The `Button`
class provides a `setOnClick()` method to add one action to be executed when
mouse click is detected. What `Button` does in `addOnClick()` is not shown, but
you can assume it stores the `onClick` object in a private variable and will
invoke it when a mouse click is detected.

In order to observe the button, we must implement `OnMouseClick` with our custom
action. Typically using a anonymous Java class:

{% highlight java %}
Button btn = new Button("Click me!");
btn.addOnClick(new OnMouseClick {
  void onClick(Control control) {
    System.out.println("You have clicked on the button!");
  }
});
{% endhighlight %}

It's time to think about why do we need this pattern. What we are doing here
is represent pure behavior using the `OnMouseClick` class hierarchy. As we have
seen in the example, we used an anonymous function that holds no internal state
to do its job. Do we really need to declare an interface for that? And do we
need to implement classes (anonymous or not) to define what to do in case of
a mouse click event?

If we think about pure OOP, the answer is yes. Classes are the mechanism to
represent behavior, and interfaces the way to deal with polymorphism. It may
sound weird to use an interface to represent a single behavior. So it does using
a class to implement it. Thus, it seems like we are fighting against OOP. We
need a mechanism we lack to represent a single action, and we twist the OOP
tools to represent something similar to that.

There is a much better abstraction to represent single actions in the system.
As Gandalf would say: use functions fools!

{% highlight scala %}
type OnMouseClick = (Control) => Unit;

class Button extends Control {
  def addOnClick(onClick: OnMouseClick): Unit = { /* ... */ }

  // More code here
  // ...
}
{% endhighlight %}

In this Scala example, the observer is not represented by an interface but a
function. We define `OnMouseClick` as an alias of any function that receives
a `Control` instance as argument and returns no value (`Unit`). Since Scala
functions are first-class citizens, you can pass them as argument to other
functions, assign them to variables, etc.

{% highlight scala %}
Button btn = new Button("Click me!")
btn.addOnClick(control => println("You have clicked on the button!"))
{% endhighlight %}

As simple as that. Thus, observer pattern provides the means to simulate
functions using objects due to the lack of the former in pure OOP. This pattern,
and others that try to encapsulate pure behavior using objects (Strategy,
Factory, Adapter...) are actually anti-OOP patterns.


## Visitor Pattern

Our third anti-OOP pattern is not as obvious as the previous ones. Mainly
because [Visitor pattern][3] is not even easy to understand. In general terms,
Visitor is used to execute an algorithm over an unknown object structure. Let's
see the following example:

{% highlight java %}
interface CarElementVisitor {
  void visit(Wheel wheel);
  void visit(Engine engine);
  void visit(Car car);
}

interface CarElement {
  void accept(CarElementVisitor visitor);
}
{% endhighlight %}

Let's start discussing these two interfaces. `CarElementVisitor` represents the
entity that visits all the parts of a car. Here _visit_ means what the actual
implementation decides. One `visit()` method is provided for each element that
can be visited.

On the other hand, `CarElement` represents part of a car structure that will be
visited, including the car itself. The `accept()` function must accept a visitor
that will be instructed how to visit the structure. We can implement this
interface as follows:

{% highlight java %}
class Wheel implements CarElement {
  public void accept(CarElementVisitor visitor) {
    visitor.visit(this);
  }
}

class Engine implements CarElement {
  public void accept(CarElementVisitor visitor) {
    visitor.visit(this);
  }
}

class Car implements CarElement {
  public void accept(CarElementVisitor visitor) {
    for (CarElement elem : this.elements) {
      elem.accept(visitor);
    }
    visitor.visit(this);
  }
}
{% endhighlight %}

For each single element, `accept()` only invokes `visit()` on the visitor. For
`Car` class, it requests each subelement to accept the visitor before letting
it visit itself.

Finally, we could provide an implementation for a `CarVisitor`, as in:

{% highlight java %}
class Mechanic implements CarVisitor {
  public void visit(Wheel wheel) { /* ... */ }
  public void visit(Engine engine) { /* ... */ }
  public void visit(Car car) { /* ... */ }
}
{% endhighlight %}

Why do we need this so complicated design? Why don't we implement `accept()` as:

{% highlight java %}
class Car extends CarElement {
  public void accept(CarElementVisitor visitor) {
    for (CarElement elem : this.elements) {
      visitor.visit(elem);
    }
    visitor.visit(this);
  }
}
{% endhighlight %}

Well, this code does not even compile. The statement into _for_ expression tries
to invoke `visit()` with `CarElement` as argument. `Visitor` implements
`visit()` for `Wheel`, `Engine`, `Car`..., but not for `CarElement`. We used
`accept()` function before to let each `CarElement` implementation choose the
appropriate `visit()` implementation that works for it:

{% highlight java %}
class Wheel implements CarElement {
  public void accept(CarElementVisitor visitor) {
    visitor.visit(this); // This invokes visit(Wheel)
  }
}

class Engine implements CarElement {
  public void accept(CarElementVisitor visitor) {
    visitor.visit(this); // This invokes visit(Engine)
  }
}
{% endhighlight %}

Perhaps you never heard about it, but this have a name. What we are simulating
here is known as [double dispatch][4], or more general [multiple dispatch][5].
You are familiarized with single dynamic dispatch. The actual implementation of
a method is selected in runtime by the target object:

{% highlight java %}
interface Greetings {
  void hello();
}

class Foo extends Greetings {
  public void hello() { System.out.println("Hello"); }
}

class Bar extends Greetings {
  public void hello() { System.out.println("Hi!"); }
}

Greetings g = /* ... */;
g.hello();
{% endhighlight %}

When `hello()` is invoked, the actual implementation is selected in base of
`this` object (`Greetings` implementation). That's single dynamic dispatch.
What's double dispatch then? The ability to choose the method implementation
based on `this` object and also one of the arguments passed to the function.
That's what the non-compiling example above was trying to show. If Java would
support double dispatch, it would be possible to invoke `visit()` by passing
a `CarElement` object, and the runtime would select the appropriate
implementation depending on the instance passed as argument.

Double dispatch is rarely found in mainstream OOP languages ([C# has support][6]
for that using `dynamic` type). One more time, the Visitor pattern is a way to
simulate the lack of support for double dispatch. It is another anti-OOP design
pattern.

Just to mention, a more elegant way to simulate multiple dispatch is a language
feature known as [pattern matching][7]. Again, Scala is a good candidate to
show an example.

{% highlight scala %}
class Mechanic implements CarVisitor {
  def visit(elem: CarElement): Unit = {
    elem match {
      case w @ Wheel => visitWheel(w)
      case e @ Engine => visitEngine(e)
      case c @ Car => visitCar(c)
    }
  }

  private def visitWheel(w: Wheel): Unit = { /* ... */ }
  private def visitEngine(e: Engine): Unit = { /* ... */ }
  private def visitCar(w: Car): Unit = { /* ... */ }
}
{% endhighlight %}

Now it is possible to have our double dispatch in `Car`:

{% highlight scala %}
class Car extends CarElement {
  def accept(visitor: CarElementVisitor): Unit = {
    for (elem <- this.elements) {
      visitor.visit(elem);
    }
    visitor.visit(this);
  }
}
{% endhighlight %}

So `accept()` method is not required for every `CarElement` but just `Car`.

## Summary

Long time ago I read [a question][8] in Stack Overflow about the difference
between Java and Javascript. My favorite answer was:

> One is essentially a toy, designed for writing small pieces of code, and
> traditionally used and abused by inexperienced programmers.
>
> The other is a scripting language for web browsers.

Compared to other programming languages, Java is a toy. And it is not a
coincidence that Java is the favorite language of OOP lovers. OOP is simple as
well. But insufficient in some cases. It is highly recommended to know the
weakness of every tool we use in our profession. And behind the so often
acclaimed OOP design patterns there is a dark side. Does it mean we should avoid
OOP? Of course not! It just means OOP might be insufficient. Use design patterns
to mitigate the effects of its lack of power. But be aware of that. And, of
course, leave your comfort zone and explore other paradigms. Your coding skills
will thank you.

[1]: https://en.wikipedia.org/wiki/Singleton_pattern
[2]: https://en.wikipedia.org/wiki/Observer_pattern
[3]: https://en.wikipedia.org/wiki/Visitor_pattern
[4]: https://en.wikipedia.org/wiki/Double_dispatch
[5]: https://en.wikipedia.org/wiki/Multiple_dispatch
[6]: http://www.codeproject.com/Articles/69407/The-Dynamic-Keyword-in-C
[7]: https://en.wikipedia.org/wiki/Pattern_matching
[8]: http://stackoverflow.com/questions/245062/whats-the-difference-between-javascript-and-java