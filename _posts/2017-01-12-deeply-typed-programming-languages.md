---
title:  "Deeply typed programming languages"
date:   2017-01-12
category: languages
---
Uncle Bob did it again. Some months ago he wrote [Type Wars][1], a post to defend static type systems are not really needed if you do TDD. Now he's back. In his latest post, [The Dark Path][2] (I know: these titles sound like Star Wars sequels) he brings new weapons for the dynamic languages enthusiasts ([argument from authority][3] included) to tell everyone else all these static type checks are useless.

In his new article, Bob tells he has been in the playground with Swift and Kotlin. And he didn't like the experience. In his own opinion, these languages are going too far (or too deep) introducing strongly typing features. And (guess what?)... who needs such strong typing when you have TDD?

After Type Wars, I wrote my own post explaining [why Uncle Bob was wrong][4]. And I feel the irresistible need to reason why he's wrong again.


## The risks

Uncle Bob is right saying there are multiple risks using many features of the programming languages. He mentions a few. An uncaught exception could cause a disaster in production. Just as an unperceived method override in case of classes _open_ by default. Or the [Billion Dollar Mistake][5] (aka. null reference). But, according to the recurrent Bob's words: **preventing them in the language is the wrong path**.

Since Uncle Bob claims we are going too far, let's go even further. But in the same direction he's trying to row.

When discussing about exceptions, open classes or nulls, he asks this question again and again:

_"Whose job is it to manage that risk? Is it the language’s job? Or is it the programmer’s job."_

Good. Now let's go back 30 years. Let's say there is a fictitious old man (Grandpa Bob), with decades of coding on this back, writing an article about his experiences using a new cutting edge language: Java.

_"In Java, you cannot manage your own program memory. Every byte of allocated memory is managed by something called Garbage Collector. For sure, you cannot even reference a variable in the stack. By God every object have to be allocated in the heap memory!_

_Now, perhaps you think this is a good thing. Perhaps you believe manual memory allocation is a source of error and risk. Perhaps you think you can eliminate memory leaks by forcing programmers to follow a memory model in which objects are freed by their own. And you may be right. Handling memory manually is a risky thing. Lot of things can go wrong when you dereference the wrong pointer._

_The question is: Whose job is it to manage that risk? Is it the language’s job? Or is it the programmer’s job."_

I just hope Uncle Bob doesn't like cooking. After all, the risk of finger cut off is a cook's job. I fear he would use knifes with just blades and no handle.

## Cost vs benefit

I think nobody in his right mind would agree garbage collector is going too deep. Memory leaks and segment violations were really dramatic errors. And we all are happy to delegate this task in the language.

Now, perhaps you think this is because garbage collector is for free. It does not slow down the programmer with stupid null checks or idiot try-catch blocks. And you may be right. For most applications, the use of garbage collection has almost zero cost. But for some use cases, that's not true. Using a garbage collector in systems software where there is no operating system or room for a complex runtime is a real problem (Rust designers know that). In the other hand, having GC prevents the language to implement the [RAII][6] pattern (good topic for a post), the most clever solution to resource managing I've ever seen. In general terms, garbage collection is cheap. Its cost is very low, and the benefit it brings is high. But that's not true for some specific cases.

And that's the way we have to evaluate whether a programming language feature is appropriate or not. What is the cost it takes and what is the benefit it brings. And both, cost and benefit, are heavily affected by the conditions under our software have to be executed and developed.

Forced null checks have a extreme cost if I make data analysis experiments. And they provide a little benefit in my 1,000 lines code I use to retrieve some CSV files, parse them and make some statistical analysis. Also, mandatory exception catching is extremely expensive in the experimental app I wrote to validate a new business model that will make me rich. And, after all, an uncaught exception is something I can afford since my users are not paying yet for the service.

In contrast, there are many other scenarios where forced null checks or mandatory exception catching are redeemed by far. From the extreme where people may die because of a NPE, to the real life of many large teams that systematically loose many salaries having half of development team fixing stupid bugs that might be detected by the compiler.

Obviously, if I'm just in the playground with a new language as Uncle Bob was with Swift and Kotlin, I'm quite far to experiment any benefit from the _depth_ of the type system. In my experience, many programmers coming from dynamic languages see no benefit on static typing because of that: in the hundred lines they wrote following the language getting start guide, they saw no benefit from the type system. So they quit.

## Humans vs computers

But, after all the benefit of type system _depth_ can be fully replaced by TDD, right? [I wrote about this fallacy][4] before, and I have no much more to say. Just one more thing. **Humans are as good performing manual tasks as computers doing creative things**. In my experience, anything that depends on human diligence and discipline fails the sooner or later. If some task is repetitive, tedious, error prone, let's the computer do it by you. There is no reason to have the programmer and his ability to write null-check tests as last resort before loosing the money of your company. Let the maths do the job for you and spend the time of your programmers creating, which is what computers can't do well.

## Conclusions

We live in a time that most programmers want to do things as fast as possible. It's really unusual to find people ready to invest. Invest time to learn how to improve their abilities. Invest time to incorporate new techniques to their tool belts. Unfortunately, there are some things that cannot be learn just reading blogs or making coding katas. They require the real experience of seeing how your compiler detects an error that otherwise would require several days debugging and fixing (not to mention the impact in money lost). For many programmers, this will be just another rant from one of these obsolete Java developers.

Uncle Bob thinks TDD is a golden hammer. He's wrong. TDD is a fantastic software quality tool to check what the type system cannot verify. To check those things that depend on our creativity and are beyond the capabilities of the compiler. Bob thinks languages that prevents the most common errors are the dark path. He's wrong. Something I learned reading _The Clean Coder_ was that professionalism is one of the most precious things we have as developers. And, as a fair professional, I want my software to work as free of bugs as possible when they have a significant impact. And TDD is not enough for that.

In summary, **I'm really proud of walking on the dark path of deeply typed programming languages.**

[1]: http://blog.cleancoder.com/uncle-bob/2016/05/01/TypeWars.html
[2]: http://blog.cleancoder.com/uncle-bob/2017/01/11/TheDarkPath.html
[3]: https://en.wikipedia.org/wiki/Argument_from_authority
[4]: {% post_url 2016-05-03-tdd-vs-static-typing %}
[5]: https://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions
[6]: https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization
