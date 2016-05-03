---
title:  "TDD vs Static Typing"
date:   2016-05-03
categories: Scala
---

Yesterday Uncle Bob posted an article about [Type Wars][1]. In summary, the post describes his own experience with different type systems across the times. How types appeared in the mainstream languages and how they competed from the early days to the dynamic vs static typing debate of our days.

There is an interesting introduction in the post about what lead to Uncle Bob to consider type systems provide a real benefit. In the early days, as an assembler coder who writes software for ancient and simple hardware architectures, types were pointless. As the systems became more sophisticated (and so its software), the benefits of type systems were obvious. Large codebases cannot be written without the aid of a type system.

In spite of this introduction, many people has interpreted that Uncle Bob is against static typing. Mainly because he affirmed that the most important thing to guarantee software quality is TDD.

Well. There is nothing more daring for a software developer than saying that Uncle Bob is not completely right. I know it. This is just a modest attempt to explain the reasons I have to consider TDD is not a replacement of static typing. Just four facts about TDD that Uncle Bob had left unsaid.


## #1: TDD is optional

I am absolutely sure that Uncle Bob code has a code coverage of almost 100%. Thanks to that, he is pretty sure his code works as expected and he is able to evolve and adapt it to new requirements.

But only Uncle Bob is like Uncle Bob. The vast majority of human beings are really far to get 100% of code coverage. Ok, I know, I know. You are of his kind. You are one of these semi-gods that get almost perfect code coverage. I heard many people asserting that before they revealed they were lying. Even so. It is very likely that your team mates are not gods of the coverage like you. And so the rest of developers in your company. And so the authors of the dozens of libraries you use in your codebase. Face it: your code is unlikely to have almost 100% coverage.

In the other hand, Uncle Bob, your team mates, any developer in your company, your libraries' authors, and you must follow the rules of the static type system if you code in such kind of languages. Otherwise your code will not compile. It will never be shipped. **TDD is optional. Static type system is not.**

Note for distracted readers: **I didn't mean** TDD is not necessary as long as you have static typing. I just said a quality technique you cannot trust is not effective.

## #2: TDD is hand-written

Why is almost complete code coverage so hard to get? There are several reasons. One of them is that some kind of tests are boring.

Yes, boring. Let's see an example. I have a class that implements a probabilistic classifier. Given one element, it tells me whether it goes to box A or box B randomly according to some probability. Something like this:

{% highlight scala %}
class BinomialClassifier(probability: Double) {
  require(probability > 0.0 && probability < 1.0)
  // ...
}
{% endhighlight %}

For sure, I want my probability to be in range (0.0, 1.0). So I write the following test case:

{% highlight scala %}
"Binomial classifier" should "accept only valid probability in its constructor" in {
  an[IllegalArgumentException] should be thrownBy new BinomialClassifier(-1.0)
  an[IllegalArgumentException] should be thrownBy new BinomialClassifier(2.0)
}
{% endhighlight %}

How many times did you had to write this kind of test cases? And null checking test cases? If the answer is hundreds or thousands, I'm sorry for you. You are wasting your time in repetitive and boring tasks. If the answer is none, I'm even more sorry. Your code is (not even dynamically) type safe.

We could avoid such repetitive tasks if the language (this time Scala) allows something like this:

{% highlight scala %}
type Probability = double in range (0.0, 1.0)
class BinomialClassifier(probability: Probability) {
  // ...
}
{% endhighlight %}

Now the compilers would work for me. The type system ensures any instance of `Probability` is in the expected range. It is fully type safe and I had to write no test case for that. Note for distracted readers: yes, Scala is a statically- and strongly-typed language. But not stronger enough to include number subtyping. In this case, the type system gives the same guarantee any other dynamic language would do.

This is a very simple scenario, indeed. But the more simpler it is, the more likely that we decide not to write a test case to cover it. In languages like Python or Ruby, how many tests did you write to check that parameters passed to the constructor are actually used to populate object attributes? None. How do you often check trivial getter functions are already defined? Never.

I know. You don't need such test cases. Because there are other tests that require these attributes and functions and they will fail if they are missing. But what if you are not Uncle Bob and you didn't write these other test cases? Everything seems to work well in spite of this part of the code is not tested. And then it comes a refactor. Some attributes and some functions are removed or renamed and then... Boom! `AttributeError`, `NoMethodError` and friends in production! Face it. You have seen this many times.

In a statically typed language, even if you didn't write such tests (which is also possible for everybody but Uncle Bob because #1: TDD is optional), after removing or renaming an attribute or a function the code that depended on them doesn't compile. This is automatically checked for you. Without hand-written test cases. For free.

## #3: TDD is not enough for refactoring

TDD is a necessary but not sufficient tool for refactoring. It is necessary, because you must ensure your business logic is not broken as you change some parts of your code. But it is not sufficient because refactoring your code without static typing is a fucking nightmare.

Static typing is not just a code checking tool. It is also the best source of information you can have about your code. Since any value, expression, object, function, has a type known at compile time, and the scope of each one is unambiguous, you may use this information to turn refactoring into a child's play.

Renaming a function is as simple as telling your IDE to do it. It will use the static type information obtained from the compiler to know where this function is invoked. It will change the name of the function were it is declared and also the name of the function in each expression where it was called. The same applies for packages, modules, classes, interfaces, traits, methods, constants, or any other building block of the language. Renaming, moving, splitting, extracting, merging. Your toolset has access to the required information to do the job without breaking a single dish.

Face it. Every time you had to do big refactors in a dynamically-typed language you felt as a bomb squad officer. Changing a class is the closer you will get to "cutting the red or the blue wire" dilemma. And not only because most of the changes must be done by hand without the assistance of any tool. Guess what? You are not Uncle Bob, and you will find parts of your code without test coverage. Any change on any line of code may lead to a dirty runtime exception in production. So open your eyes as much as you can and cross your fingers.

## #4: TDD and static typing complement each other

If static typing is so awesome, is it possible to get rid of TDD in favor of the compiler? Absolutely no. There is something that will be out of scope of the type system: the business logic of your program.

The programming language provides building blocks: values, expressions, objects, classes, branches, loops, functions, ... . It also provides a set of rules that determine how the building blocks can be composed to create abstractions that implements your business logic. Rules like "the if statement expects a boolean expression to evaluate" , "an string value only can be concatenated with expressions of string type" or "the method invocation expects an object that implements it", among many others. When the language is statically typed, TDD is not needed to detect violations of these rules. The compiler does it for free. In contrast, **for dynamically-typed languages TDD is the last resource** to check these rules are not violated. And, as we discussed above, it is not a resource you may trust. It cannot be expressed better than this tweet:

<div align="center">
<blockquote class="twitter-tweet" data-lang="es"><p lang="en" dir="ltr">“TDD replaces a type checker in a dynamically typed language in the same way that a bottle of whisky replaces your daily problems“</p>&mdash; Matt Gumbley (@mattgumbley) <a href="https://twitter.com/mattgumbley/status/727140296912441345">May 2, 2016</a></blockquote> <script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

On top of type system rules, we define the business logic of our application. Such logic defines its own rules like "an username cannot contain blank spaces", "the user must have permissions to access this resource" or "the missile must not explode before reaching its target". The best way we have to check business logic rules is TDD. I say the best and not the perfect way. Not perfect because we fail to have an almost complete code coverage. Because some tests are boring to code. If you have a good replacement to TDD, I'm all ears.

Let's say this a summary. **TDD and static typing complement each other**. TDD simply cannot replace a compiler, as the compiler cannot replace TDD.



[1]: http://blog.cleancoder.com/uncle-bob/2016/05/01/TypeWars.html
