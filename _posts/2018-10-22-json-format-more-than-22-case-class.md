---
layout: post
title: Using Jsonx w/ Case Classes Having > 22 Members
author: Fahad Siddiqui
date: "2018-10-22 16:58:00"
---

![](/assets/posts/scala-jsonx.png)

Quoting [Underscore.io](https://underscore.io/blog/posts/2016/10/11/twenty-two.html):

> Back in 2014, when Scala 2.11 was released, an important limitation was removed: “Case classes with > 22 parameters are now allowed”. This may lead you to think there are no 22 limits in Scala, but that’s not the case. The limit lives on in functions and tuples.

We recently ran into a problem where we needed to define an `OFormat` (`play.api.libs.json.OFormat`) for a case class having more than 22 members.

We've found a pretty neat [solution](https://github.com/xdotai/play-json-extensions) solving this problem.

First of all let's look into the problem,

```scala
case class Foo(a1: String, a2: String, a3: String, a4: String, a5: String,
               a6: String, a7: String, a8: String, a9: String, a10: String,
               a11: String, a12: String, a13: String, a14: String, a15: String,
               a16: String, a17: String, a18: String, a19: String, a20: String,
               a21: String, a22: String)

object Foo {
  implicit val f: OFormat[Foo] = Json.format[Foo]
}
```

This works fine. But when you add one more member to the case class `Foo`, you'll get an error:

```
Error:(.., ..) No unapply or unapplySeq function found for class A: <none> / <none>
  implicit val f: OFormat[Foo] = Json.format[Foo]
```

To solve this, `xdotai`'s repository `play-json-extensions` provides an elegant solution.

Instead of using `Json.format[Foo]` (`play.api.libs.json.Json`), you can now use `Jsonx.formatCaseClass[Foo]` (`ai.x.play.json.Jsonx`) which supports format for case classes more than 22 members.

Summing up,

```scala
import ai.x.play.json.Jsonx
import play.api.libs.json.{Json, OFormat}

case class Foo(a1: String, a2: String, a3: String, a4: String, a5: String,
               a6: String, a7: String, a8: String, a9: String, a10: String,
               a11: String, a12: String, a13: String, a14: String, a15: String,
               a16: String, a17: String, a18: String, a19: String, a20: String,
               a21: String, a22: String, a23: String)

object Foo {
  implicit val f: OFormat[Foo] = Jsonx.formatCaseClass[Foo]

  def empty(): Foo = {
    this("", "", "", "", "",
         "", "", "", "", "",
         "", "", "", "", "",
         "", "", "", "", "",
         "", "", "")
  }
}

object MainObject {
  def main(args: Array[String]) = {
    println(Json.toJson(Foo.empty()).as[Map[String, String]])
  }
}
```

works perfectly fine!
