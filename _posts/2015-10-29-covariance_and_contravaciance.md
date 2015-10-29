---
title:  "Covariance and Contravariance"
date:   2015-10-29
categories: typing
---

Some weeks ago I gave a introductory talk of Scala for Java programmers. At
some point I introduced algebraical data types in Scala using sealed traits,
and I though it was a nice moment to show how the language supports covariance
and contravariance for generic types.

The audience didn't agree it was a nice moment. They weren't familiarized with
these terms at all, and we had no time enough to discuss the point in depth.
So I decided it was a very good topic to discuss in typeinference.com.

Any programmer is familiarized with the subtyping relationship between two
types. Let's say we have `Base` and `Derived` classes so `Derived` inherits
(or extends) `Base`, as in:

{% highlight java %}
class Base { ... }
class Derived extends Base { ... }
{% endhighlight %}

The inheritance mechanism conveys a subtyping relationship of `Derived` respect
`Base`. In other words, any instance of `Derived` is also an instance of
`Base`. Because of that, the following statements are valid in Java.

{% highlight java %}
Derived foo = new Derived(...);
Base bar = foo;
{% endhighlight %}

All right. Now let's say we have `List<Base>` and `List<Derived>` instead. Is
there any subtyping relationship among them? Is the following code valid?

{% highlight java %}
List<Derived> foo = new ArrayList<Derived>();
List<Base> bar = foo;
{% endhighlight %}

I'm sure your brain says _"Sure! Why not?_" but your instinct prevents you from
answering. For sure, things get complicated with generics, and this kind of
subtyping relationships have non trivial consequences.




## Java generics are not covariant

The property of a complex type as `List<Derived>` to be a subtype of
`List<Base>` is known as covariance. And it is not supported in Java
collections. Therefore, the code above compile with errors.

You may think it is Ok to consider a collection of `Derived` as subtype of a
collection of `Base`. After all, as they are collections, they could be seen
as a subset (in algebra of sets) of elements of that type. Since `List<Base>`
is a subset of `Base`, `List<Derived>` is a subset of `Derived`, and `Derived`
is a subset of `Base`, any `List<Derived>` must be also a subset of `Base`, and
therefore a subset of `List<Base>` as well. And this rationale is correct, so
covariance is natural for collections.

But, if maths say we are right, why do Java prevents us to use covariance?
Because of this:

{% highlight java %}
class OtherDerived extends Base { ... }

List<Derived> foo = new ArrayList<Derived>();
List<Base> bar = foo;
bar.add(new OtherDerived(...));
{% endhighlight %}

Let's say we have `OtherDerived` class that extends `Base` as `Derived` does.
We use the `bar` variable, of type `List<Base>`, to insert a new element
of `OtherDerived` type. This is valid, since `List<Base>` may contain any
instance of `Base`, including those of `OtherDerived` type. But remember: `bar`
is a reference to an instance of `ArrayList<Derived>` class. Any element
inserted in `bar` would become also an element of `foo`. Inserting such a
`OtherDerived` instance in `bar` means `foo` contains elements of `Derived` and
`OtherDerived` types. And that is contrary to the type contract of `foo`
(`List<Derived>`).

This is a good reason for Java to consider generic types non covariant.
Nevertheless primitive arrays **does support** covariance, making the following
code valid to the compiler.

{% highlight java %}
Derived[] foo = new Derived[...];
Base[] bar = foo;
bar[0] = new OtherDerived(...);
{% endhighlight %}

Although it compiles, it is not type safe and it would crash at runtime with
an exception. The reason to support covariance for primitive arrays is likely
related with the sorting functions of `java.util.Arrays`. Or perhaps they
didn't realize they were type unsafe until they were implementing the JVM. Who
knows?


## Covariance, contravariance and functions

Covariance is closely related to functions. Actually, the problem discussed
above may be seen from the perspective of `add()` method and how the
covariance affects subtyping.

Perhaps you didn't think about it, but functions are also types. And, as such,
they may have subtyping relationship among them. For instance, the following
two functions keep a subtyping relationship.

{% highlight java %}
public Base foo(Base value);
public Derived bar(Base value);
{% endhighlight %}

Here `bar` is a subtype of `foo`. Following [Liskov substitution principle][1],
wherever a function `Base -> Base` is required, you may use the function `Base
-> Derived`. That is natural: if the function is invoked expecting a `Base` as
result, receiving a `Derived` instance is also valid since `Derived` objects
are also `Base` objects.

That is respect the return value. What about function arguments? This is where
we introduce a new concept very hard to explain and understand: contravariance.
So please read the following lines carefully.

Let's say we have the following two functions:

{% highlight java %}
public Base foo(Base value);
public Base bar(Derived value);
{% endhighlight %}

Is `bar` a subtype of `foo` as before? To be so, wherever we expect a function
`Base -> Base` we should accept a function `Derived -> Base`. If we do so, we
could invoke such function with an instance of `Base` as input argument. But
`Derived -> Base` cannot accept that! It needs a `Derived` instance. So indeed
`bar` is not a subtype of `foo`.

And what about the other way around? Is `foo` a subtype of `bar`? Again, to be
so wherever we expect a function `Derived -> Base` we should accept a function
`Base -> Base`. And this time... that's true!!! If we invoke such function with
a `Derived` instance as input argument, `Base -> Base` would accept it since
`Derived` objects are also `Base` objects. So `foo` is a subtype of `bar`
indeed.

If we represent unary functions using generics, we might have a type like
`Function<T, R>`, where `T` is the type of the input argument and `R` is the
type of the function result. According to the subtyping relationships we just
have illustrated:

* `Function<T, Derived>` is a subtype of `Function<T, Base>`. As we discussed
before, `Function<T, R>` is covariant respect `R` parameter.
* `Function<Base, R>` is a subtype of `Function<Derived, R>`. This is exactly
the opposite to covariance, so we say `Function<T, R>` is contravariant respect
`T`.

All this leads to the following rule: functions are **contravariant in the input
type** and **covariant in the output type**.

## Covariance, contravariance and inheritance

In order to one class to extend another class, it should provide all its members
to the same type. That comprises its methods. So in:

{% highlight java %}
class Foo {
    public Base method(Derived value) { ... }
}

class Bar extends Foo { ... }
{% endhighlight %}

... the `Bar` class should override `method` to the same function type as
defined in `Foo` signature. The same function type, including a subtype
function. So the following code is correct.

{% highlight java %}
class Foo {
    @Override
    public Derived method(Base value) { ... }
}
{% endhighlight %}

Going back to Java `List<T>` discussion, we may demonstrate it is not covariant
because of `add()` method from the function subtyping perspective.

{% highlight java %}
class List<T> {
    public T get(int i) { ... }
    public void add(T value) { ... }
}
{% endhighlight %}

If we instantiate `List<T>` with `Base` and `Derived`, we would have something
equivalent to the following classes:

{% highlight java %}
class BaseList { // instantiated as List<Base>
    public Base get(int i) { ... }
    public void add(Base value) { ... }
}

class DerivedList { // instantiated as List<Derived>
    public Derived get(int i) { ... }
    public void add(Derived value) { ... }
}
{% endhighlight %}

In order to check whether `DerivedList` is a subclass of `BaseList`, we must
check its function. Respect `get()` method, there is no problem at all.
`int -> Derived` is a subtype of `int -> Base`, so covariance rules are not
violated. Respect `add()` function, `Derived -> void` is not a subtype of
`Base -> void`, so covariance is broken.

## Covariant generics and immutability

Although Java doesn't support it, covariance is compatible with generic types.
With the appropriate compiler support (as Scala compiler provides), it would
be possible to specify constraints on the generic parameters to avoid
situations where type integrity is broken.

The following code shows how we can create a generic class `List[T]` in Scala
that supports covariance.

{% highlight scala %}
class List[+T] {
  def get(i: Int): T = { ... }
}
{% endhighlight %}

In this code, `[+T]` means the generic class is covariant respect `T`, so the
following code is valid:

{% highlight scala %}
val foo: List[Derived] = new List
val bar: List[Base] = foo
{% endhighlight %}

All right. But we are cheating here. As discussed above, `get()` method is not
an obstacle for covariance as `add()` is. If we try to add the latter we
would face problems.

{% highlight scala %}
class List[+T] {
  def add(value: T): Unit = { ... }
}
{% endhighlight %}

This code compiles with the following error:

{% highlight text %}
error: covariant type T occurs in contravariant position in type T of value value
       class List[+T] { def add(value: T): Unit = ??? }
{% endhighlight %}

Scala compiler knows covariant parameters cannot be used as input arguments,
which are in a contravariant position. We can fix that by using a generic
function instead.

{% highlight scala %}
class List[+T] {
  def add[U >: T](value: U): List[U] = { ... }
}

val foo: List[Derived] = new List
val bar: List[Base] = foo

val newBar: List[Base] = bar.add(new Base)
val newNewBar: List[Object] = newBar.add(new Object)
{% endhighlight %}

Using this `add()` method, the type of `value` is constrained to be a
supertype of `T`. That's what `[U >: T]` means. Also, instead of returning
`Unit` (e.g., `void`), we return a new list resulting from appending `value`
and the end of the target list. In other words, we are making our `List<T>`
type immutable. Why do we do that? Well, it is not easy to explain, but if we
keep some storage to hold the elements of the list, this storage would hold
elements of type `T`. You cannot add elements of type `U` to that storage.
The only possible option is to return a new list instance that considers all
its members as `U` type. So inserting a `Base` value in `bar` returns a new
list of `Base` objects (previous elements of the list were `Derived` objects,
which are also of type `Base`). We could even add an `Object` element to that
list, which produces a `List<Object>` list.

## Conclusions

Since we are discussing about algebra of sets, let's make a new allegory using
sets. Consider a set `P` of all possible computer programs. There is a subset
of `P`, you call it `C`, that represents the programs your compiler considers
right (compile without errors). There is another subset of `P` known as `V`
that represents the valid programs, i.e. those programs that work according to
the desire of their authors.

In a perfect world, `C` and `V` would be exactly the same set. In other words,
your compiler would compile without errors every valid program. And every
program your compiler accepts would be valid and would have no errors. But
this happens only in a perfect world. In the real world, your compiler
considers some valid expressions (from the type system perspective) as errors.
And of course, some of the programs your compiler doesn't complain about are
wrong.

Covariance and contravaciance are type system features some compilers don't
cover. This means some valid programs are not accepted by these compilers. You
might think such features are barely useful, and they doesn't worth the effort
of understanding their operation. But, if you think about them in `C` vs `V`
terms, you would realize we are increasing the size of `C`, conquering new
lands to `V`. More and more valid programs would be accepted by your compiler.
The intersection of `V` and `C` would be reduced. And that's indeed a real
benefit.

[1]: https://en.wikipedia.org/wiki/Liskov_substitution_principle
