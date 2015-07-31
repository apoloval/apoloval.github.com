---
title:  "Rust Data Ownership, Part I"
date:   2015-07-31
categories: Rust
---

[Rust][1] is one of the most interesting things I discovered this year. After
long time programming in C++, I had hundred of situations where its memory
management model drove me mad. Even using smart pointers, it is really hard to
reason about who has the ownership of every object. If you ever wrote code
using Boost ASIO with C++11 lambdas you know what I mean.

I also tried some other alternatives to C++. [D][2] was promising, but its
garbage collector leaves no place for [RAII][3] (one of my favorite idioms).
[Go!][4] uses a garbage collector as well, and it also has a poor integration
with legacy systems. So I continued programming in C++ with a sight of
resignation.

Some day a friend of mine told me about Rust. I took a look with caution. I
read [their book][5], code some examples. The more I learned the more I was
convinced I had something really great in front of me. Something that could lead
me to stop writing any further line of code in C++.

I've been wishing for some time to write about how Rust solves the memory
management problem in a very clever way no other mainstream language ever tried.
And I'd like to do it by comparing this mechanism with the one provided by
C++11. And there is a lot of things to discuss! This is the first part of a set
of posts about this topic. Today we are gonna see a basic introduction to data
ownership and movement semantics.



## Ownership

Many of us learned the programming principles so long time ago that we have
stopped thinking about ownership. And garbage collectors hasn't contributed to
recall. In many high level lenguages we access our program data through
variables. A variable is a name, a binding to a single piece of data. Some
examples in C++:

{% highlight cpp %}
int a = 10;
int b;
std::string c = "foobar";
{% endhighlight %}

Here one instance of `10` is bounded to the variable `a`, while another
instance of `std::string` type (with `"foobar"` content) is bounded to `c`.
What about `b`? Is that an exception to the rule? No! `b` has no explicit
value, so it takes a `0` according to C++03 specification. Rust uses a similar
syntax:

{% highlight rust %}
let a = 10;
let b: u64;
let c = "foobar".to_string();
{% endhighlight %}

Well, perhaps it is not *that* similar. Rust implements something known as
[type inference][6]. Type inference is a really good name for a coding blog,
but it is also the ability of a compiler to infer the type of a variable,
function argument, function result, etc, from its context. Here, since
we are assigning the number `10` to `a`, the compiler infers `a` is an integer.
For `b` we had to specify its type explicitely. This is obvious, if we only
declare `let b`, the compiler lacks enough information to infer what its
type is. But let's forget about type inference for now and come back to the
ownership.

Let's say we add a new declaration like this one:

{% highlight cpp %}
std::string c = "foobar";
std::string d = c;
{% endhighlight %}

What does it exactly mean? `c` was bounded to a new string `"foobar"` as before.
And then `d` was bounded to... `c`? Not really. You are likely thinking about
*copy*. The content of `c` is copied into `d`, so now each one is bounded to
different but equal data. That's correct. But now... forget about strings. Let's
say `c` type is something that cannot be copied. Something like:

{% highlight cpp %}
std::thread c = ...;
std::thread d = c;
{% endhighlight %}

In C++11, the `thread` class of standard library cannot be copied. That makes
sense (at least to me). So this code doesn't compile. Now, let me suggest
the following change.

{% highlight cpp %}
std::thread c = ...;
std::thread d = std::move(c);
{% endhighlight %}

Without entering into details about `std::move()` function (which is one of
the most bizarre things introduced by C++11), let's just say that this code
is moving the contents of `c` into `d`. This is where things became interesting.
In previous examples, every variable has the ownership of the data it holds.
`a` owns `10`, `b` owns `0`. But this rule is now broken by this movement
feature introduced by C++11. The ownership of `c` is transferred to `d`. So
`c` is not the owner of the thread anymore. Actually, it has lost the access
to the thread.

The movement semantic is also present in Rust. Let's see the following example.

{% highlight rust %}
let c = "foobar".to_string();
let d = c;
{% endhighlight %}

This code is almost the same we had for C++ string variables. And we said
`c` was copied into `d`. Right? Mmmm... Right???

Not really. Rust defines itself as a systems programming language. Speed is
a major goal. And copying strings from here to there blithely is not precisly
what its authors consider speed. Here Rust assumes that `c` is moved (not
copied) into `d`.

Let's come back to C++ again for a while. Well, we said we have movement
semantics, right? We have seen how to apply them to `std::thread` type, but
most copyable types also supports movement. So, as we move threads from one
variable into another, we can also move strings.

{% highlight cpp %}
std::string c = "foobar";
std::string d = std::move(c);
std::cout << c << std::endl;
{% endhighlight %}

Place your bets! What's exactly printed in standard output? Well, we are quite
sure that `"foobar"` is not a good candidate. We said `c` is moved into `d`.
If `c` is still `"foobar"` we had copied it, not moved. So, what the hell is
into `c`?

Believe it or not, but `c` is `""` (empty string). Why? Well, it's hard to
say. It has an empty string because that value is as good as any other. If
you move the contents from `c`, it should have a neutral, none, nill, zero
value. Actually you should not use `c` anymore. You stripped the data
ownership off him. It is soulless, empty. Nevertheless, the language must
specify the behavior when, after movement, that variable is accessed again.

In C++, the movement constructor of each type decides what's the state of a
moved object. `std::string` constructor decides to leave an empty string
behind. But, what about other types? E.g., `std::list` leaves an empty list. In
general, most types decide to replace the state by a neutral value. But, what
about that types that has no natural neutral value? Could you tell what's the
neutral state for `std::thread`? Or `std::promise`? `std::future`? This problem
leads to one of the most dark sides of C++11. In [cppreference.com][7], you have
to read bizarre things about `std::thread` constructor like:

> `thread();` (1)	(since C++11)
>
> `thread( thread&& other );` (2)	(since C++11)
>
> Constructs new thread object.
>
> 1) Creates new thread object which <font color="red">does not represent a 
thread</font>.
>
> 2) Move constructor. Constructs the thread object to represent the thread of
execution that was represented by `other`. After this call `other`
<font color="red">no longer represents a thread of execution</font>.

So I have the meanings to instantiate a `std::thread` that... is not a thread
at all!

At this point, I hope you are begining to believe move semantics are dangerous.
Now let's take a look to how Rust deals with the same situation.

{% highlight rust %}
let c = "foobar".to_string();
let d = c;
println!("{}", c);
{% endhighlight %}

You feed up your compiler with this code and...

	foo.rs:4:16: 4:17 error: use of moved value: `c`
	foo.rs:4 println!("{}", c);

OMG! Is that possible? Sure! Rust compiler detects that you have moved `c`
into `d`. Thus, `c` is not valid any longer. Any attempt to access
`c` derives in a compilation error. Data ownership is fully moved, and `c`
is in a real empty state. You don't have to specify the state of the instance
after movement. You don't even have to specify the movement behavior! If
possible, the compiler will leave the data in the same memory region. If not,
its contents will be `memcpy`ied to the new location.

Move semantics could not be so dangerous after all!

This is just the begining. The way the movement semantic is implemented has
many implications. What about heap memory? How does this `memcpy`-based copy
deals with that? And what about pointers? All this and more, will be discussed
in the next parts.

[1]: http://rust-lang.org/
[2]: http://dlang.org
[3]: https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization
[4]: https://golang.org
[5]: http://doc.rust-lang.org/nightly/book/
[6]: https://en.wikipedia.org/wiki/Type_inference
[7]: http://en.cppreference.com/w/cpp/thread/thread/thread

