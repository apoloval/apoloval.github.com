---
title:  "Type Inference"
date:   2015-10-05
categories: typing
---

In a blog titled Type Inference, it's mandatory to talk about type inference.
For some time, I assumed the concept should be well known for any programmer.
That's why I thought I was a great name for a blog about programming.  But
recently I realized I was wrong. Not meaning in the blog name, which is
absolutely cool. I was wrong about the spread of type inference concept.

In the modern software industry, there are many developers that are tied to
dynamically typed languages. Python, Javascript, Ruby, NodeJS or any other
trend that is cool just because everybody says so (fashion). When you show your
doubts about the convenience of coding large scale systems using scripting
languages, they look at you as if you were a time traveler coming from the last
century. After that, most of them say _but, do you really want to code in so
much verbose languages like Java?_

After blinking twice, you tell them about type inference. And then you realize
they are not familiarized with the term at all. After a short introduction to
the concept, you point out their concept of static typing **does** come from
the last century. Every time you talk about dynamic typing I'm not thinking on
Basic. So please do not relate static typing with terrible inventions like Java.



## Why variable types must be defined

Since dynamic programming languages does not specify types until runtime
(another common mistake is to consider there are no types: wrong), they don't
need verbose statements:

{% highlight python %}
message = "Hello World!"
{% endhighlight %}

In this Python example, when this code is executed a new variable `message` is
defined with the initial value `"Hello World!"`. The variable is allocated at
runtime, and both its internal value and its type information are then bounded
to that name. Everything, from the room this value occupies in memory to what
methods it responds to are calculated at runtime.

Simplifying things, we could imagine the name 'message' is registered in some
global hash map maintained by the interpreter. The value of that map entry is,
in turn, another hash map that comprises the fields (e.g. `characters`) and the
methods it implements (e.g., `uppercase()`). When a value is used in the code,
the interpreter uses type checking approach known as [duck typing][1]:

{% highlight python %}
def foobar(message):
    other_msg = message.uppercase()
    ...
{% endhighlight %}

Here, `message.uppercase()` is evaluated by searching a `uppercase` attribute
defined in `message` hash table. If found, such attribute would be interpreted
as a function and it will be invoked. That's what makes `message` a string: it
has the attributes defined for that type. If you pass a value of any other type
that also defines a `uppercase()` method, the `foobar()` function would work
well.

Everything is dynamic. Thus, it is absolutely pointless to specify a type for a
variable or function argument in the code.

Back to the static typing world, things are a little different:

{% highlight java %}
String message = "Hello World!";
{% endhighlight %}

As you can see, in languages like Java it is common to associate an explicit
type to each variable. Since they are statically typed, they use such type
information at compile time to provide an absolutely different approach to
represent values. Instead of using inefficient hash maps to represent values,
it simply stores the fields in memory sequentially in a layout known by the
compiler. Accessing some field or invoking a method is just a matter
of reading the bytes at a determined offset from the memory location where
the value is allocated. That's much faster than resolving fields in a hash map.
But that requires that the type of each variable is defined at compile time.

## Why variable types can be omitted

Nevertheless, dynamic typing fanboys are right. There are just a few things more
stupid in programming than this:

{% highlight java %}
String message = "Hello World!";
{% endhighlight %}

Or even worse:

{% highlight java %}
MyVeryLongEntityName foo = new MyVeryLongEntityName(...);
{% endhighlight %}

The reason is simple: redundancy. If compiler is clever enough, it can **infer**
(deduce) the type of `message` easily. Since the right hand expression of the
assignment is a string value, the variable declared at the left hand must be a
string as well. That is possible in many static programming languages:

{% highlight rust %}
// In Rust
let message = "Hello World!";
{% endhighlight %}

{% highlight c++ %}
// In C++11
auto message = "Hello World!";
{% endhighlight %}

{% highlight scala %}
// In Scala
val message = "Hello World!"
{% endhighlight %}

{% highlight haskell %}
-- In haskell
let message = "Hello World!"
{% endhighlight %}

All them are statically typed languages. And in all them variable type can be
omitted.

## Type inference in functions

Type inference is not limited to the type of variables. Everywhere where a type
must be specified, there is a chance for the compiler to infer the type.

A very common case is the return type of a function.

{% highlight scala %}
// In Scala
def encode(str: String) = str.toUppercase
{% endhighlight %}

{% highlight c++ %}
// In C++14
auto encode(const std::string& str) {
    return std::toupper(str, ...);
}
{% endhighlight %}

Please note that, by contrast, inferring the type of function arguments is not
trivial. The type of the result can be inferred from the expression passed to
the return statement. But the type of function arguments cannot be easily
inferred from the function body. The compiler can analyze how the argument is
used in the function body (e.g. what methods or fields are accessed), but there
could be infinite types that implement such methods or provide such fields.
In some forms of static typing, [parametric polymorphism][2] (generics) allow
some form of type inference for function arguments. But that's another story.

## Even smarter type inference

Some compilers are smarter than others. Even on those supporting type inference,
the limits and constraints of the inference vary from one language to other.

Let's say we have the following Scala code block:

{% highlight scala %}
var a;
// more code here
{% endhighlight %}

This code clearly doesn't compile. Since `a` is not initialized to any value,
the compiler cannot infer its type.

But, what about the following code?

{% highlight scala %}
var a;
// more code here
a = "Hello World!"
{% endhighlight %}

Now the compiler does have enough information to infer that `a` is of string
type. Nevertheless, the code still invalid. Scala does not support such kind
of type inference. Rust compiler is much more clever, and supports this kind
of constructs.

{% highlight rust %}
let a;
// more code here
a = "Hello World!"
{% endhighlight %}

## Conclusions

Please stop arguing static typing sucks because of verbosity. In languages that
are not designed for Minions, the source code could be as brief as any other
dynamic language. Actually, if you are not familiar with them, languages like
Scala or Rust could be confused with scripting languages. So such claim is not
a solid reason at all.

In another post, we could discuss about some other false allegations in favor
of using dynamic typing languages for everything. Like _why do I need a compiler
if I have unit tests_ or _a REPL is absolutely essential for me_. Or my
favorite one: _I need to design my code without the compiler bothering me_.

[1]: https://en.wikipedia.org/wiki/Duck_typing
[2]: https://en.wikipedia.org/wiki/Parametric_polymorphism
