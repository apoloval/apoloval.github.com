---
title:  "Algebraic Types and Rudimentary Coding"
date:   2016-09-15
category: languages
permalink: /languages/:year/:month/:day/:title.html
---

This week I tweeted a fragment of code showing how to declare an algebraic type to enumerate the operating systems supported by your application in Scala. Surprisingly, there was some negative replies against this practice. Some people pointed out this was over-engineering, defending the code should be simpler (IMHO, rudimentary).

Tweeter is probably the worst format to discuss the proposal and explain why this is the right direction. I tried by email with no better results. Let's try with a blog post, at least to leave a proof of why I do (and will continue doing) things like that.


## The code

I'm writing a portable desktop application in Scala that should work at least in Windows and Mac OS X. One of the tasks to do it is able to store the application config and some other data files in the resources directory of the user. If you are familiarized with this problem, you will know such a directory differs from one operating system to other.

* In Windows, you'd typically store it in `C:\Users\Me\AppData\Roaming\MyApp`. The precise location might differ depending on the username and installation, so it's good to use `%APPDATA%` environment variable.
* In OSX, the most common option is `/Users/Me/Library/Application Support/MyApp`. Again, it's better to check `$HOME` env var and compute subdirectories accordingly.
* In Linux, we would use `/home/Me/.MyApp`. Again, we would use `$HOME`.

So I started to define an object in my Scala code to calculate the appropriate `Path` to store user resources for the operating system. My first attempt was something like this:

{% highlight scala %}
object UserPaths {

  lazy val Resources: Path = systemDependentResourcesPath(
    System.getProperty("os.name"))

  private def systemDependentResourcesPath(osName: String): Path =
    osName match {
      case "Mac OS X" => ???
      case "Windows" => ???
    }
}
{% endhighlight %}

Then I realized simply reading the JVM property `os.name` could be problematic. I would be coupling my `UserPaths` object to the way I detect the operating system. Using a different way (e.g. passing such information in the building process) would require changes in the `UserPaths` object, and potentially in any other part of the code that also implements a different behavior for each platform. Two of the most basic principles in software development: [encapsulation][1] and [separation of concerns][1].

So it was pretty clear that this string value and the way it is obtained should be promoted to its own type. The perfect situation to use an [algebraic type][3]. This is as simple as:

{% highlight scala %}
sealed trait OperatingSystem {
  def name: String
}

object OperatingSystem {
  case object MacOSX extends OperatingSystem {
    override def name = "Mac OS X"
  }

  case object Windows extends OperatingSystem {
    override def name = "Windows"
  }

  val Supported = Seq(MacOSX, Windows)

  def apply(): OperatingSystem = {
    val osName = System.getProperty("os.name")
    Supported
      .find(_.name == osName)
      .getOrElse(throw new IllegalStateException(
        s"unexpected operating system '$osName'"))
  }
}
{% endhighlight %}

The code is quite self descriptive. The sealed trait declares the abstract operations of an `OperatingSystem` (by now, just give its name although not necessary for this use case). Two possible values (and only two due to the _sealed_ mark): `OperatingSystem.MacOSX` and `OperatingSystem.Windows` (Linux could be added later if necessary). The method `OperatingSystem.apply()` (or shortly `OperatingSystem()`) returns the OS of the running system by checking the JVM property `os.name`.

After the three minutes required to write this code (two if you don't loose your time tweeting it), you can use it as follows:

{% highlight scala %}
object UserPaths {

  lazy val Resources: Path = systemDependentResourcesPath(
    OperatingSystem())

  private def systemDependentResourcesPath(os: OperatingSystem): Path =
    os match {
      case OperatingSystem.MacOSX => ???
      case OperatingSystem.Windows => ???
    }
}
{% endhighlight %}

## What you get from this algebraic type

Let's summarize what you get by declaring an `OperatingSystem` type in this particular use case.

As mentioned above, the way the OS is detected is not coupled to the way you compute the system dependent paths. Encapsulation and separation of concerns FTW. I'm sure the reader will be familiarized to this: your code evolves safer, the impact of the changes is extremely limited, etc, etc. This is really elementary to be discussed here.

The most important fact is that, thanks to the algebraic type you can make a [closed-world assumption][4]. `OperatingSystem` can be just one of `MacOSX` or `Windows`. Any other value is impossible. This is pretty interesting when using the pattern matching in `UserPaths` object. In the first attempt, the compiler will warn you saying the matching is incomplete. Along `"Mac OS X"` and `"Windows"` patterns, there are literally infinite elements that could be received as input. And if you want to make things right, you should protect this unit from the case a invalid or unknown string is passed as argument:

{% highlight scala %}
  private def systemDependentResourcesPath(osName: String): Path =
    osName match {
      case "Mac OS X" => ???
      case "Windows" => ???
      // Put this of expect dragons
      case other => throw new IllegalArgumentException(
        s"unknown operating system $other")
    }
}
{% endhighlight %}

In the second attempt, the compiler knows that `OperatingSystem` input is fully covered by all the patterns. You don't have to protect this part of the code against unexpected OS value.

## Criticism

Over-engineering was the word. And that's something to be worried about.

In our times many programmers consider unnecessary to evaluate `"3"+2` as an error instead of `"32"`. In a discipline whose professionals consider themselves _senior_ with barely 5 years of experience, more and more techniques that derive in unsafe and buggy code are promoted.

No, folks. A simple `String` type is not right to represent an operating system. This type have infinite possible values that are not a valid operating system name. Using it all across your code is error prone. Once you detect the OS name is invalid in some part of the code, any other function that have handled such value is suspicious to be causing the damage. Such unexpected string value may come from a hardcoded literal (let's pass `"foobar"` here and I will replace it by the good value later), wrong variable name (oh fuck! I passed `configFileName` instead of `osName` to that function!) or even an unimaginable transformation (oh! Somebody has concatenated a suffix in the `osName` variable by mistake!). In contrast, your algebraic type ensures once instantiated only can transport a valid operating system that cannot be muted or corrupted. There is only once place where it may fail: in its construction.

This is just a very simple use case of creating the right abstractions for your code. Obviously, the chance of fucking up with the OS name while calculating the user paths is really low (but not zero). But think about more complex situations. You receive a request from your API in your splendid server code, and instead of converting the bytes received from the network into a data type that only can transport a valid state... you decide to avoid over-engineering by using a more rudimentary data type that admits illegal values. Then somewhere in the code a non-possible request value appears (likely detected when running in production). Just for refusing to invest three minutes to declare the appropriate data type.

## Conclusions

In modern programming languages, defining abstractions is really cheap. Doing so you can use the type system to introduce restrictions according to what you are modeling. The cost-benefit ratio is probably the most profitable you will obtain while developing software. Don't make your client/company loose money just because you had a bad feeling or obsession with "lean" code.

And one final human factor. As the number of followers increase, more people will observe your work. Some observations will be clever. Some others not. Ignore those you cannot learn from AND you cannot make others to learn.

[1]: https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)
[2]: https://en.wikipedia.org/wiki/Separation_of_concerns
[3]: https://en.wikipedia.org/wiki/Algebraic_data_type
[4]: https://en.wikipedia.org/wiki/Closed-world_assumption
