---
layout: post
category: web micro log
tags:
---

This is just some notes on my way to learning scalaz. Of course I got help from the actual tutorial...[learning scalaz](http://eed3si9n.com/learning-scalaz).

The contents are based on my knowledge and as such is most likely wrong. Please message me any mistakes!

# Step 1

Assuming you have scala and sbt installed, to launch scalaz in your console simply create a file called `build.sbt` with the following lines [thanks stackoverflow](http://stackoverflow.com/a/19449717)):

```scala
scalaVersion := "2.11.2"

val scalazVersion = "7.1.0"

libraryDependencies ++= Seq(
  "org.scalaz" %% "scalaz-core" % scalazVersion,
  "org.scalaz" %% "scalaz-effect" % scalazVersion,
  "org.scalaz" %% "scalaz-typelevel" % scalazVersion,
  "org.scalaz" %% "scalaz-scalacheck-binding" % scalazVersion % "test"
)

scalacOptions += "-feature"

initialCommands in console := "import scalaz._, Scalaz._"

initialCommands in console in Test := "import scalaz._, Scalaz._, scalacheck.ScalazProperties._, scalacheck.ScalazArbitrary._,scalacheck.ScalaCheckBinding._"
```

# Step 2

We can get this running in the console now just by launching it using `sbt console`

# A quick look at monoids...

Monoids are semigroups, which makes it interesting for certain class of problems (from my limited understanding, useful in the map-reduce sense).

## The Monoid components

To make a monoid object we have to define two things, `mappend` which is our operator and `mzero` which is our identity function.

The hard bit is always thinking about what is the identity and the form of the set of numbers we are interested in. For example here is my implementation of average:

```scala
object avgMonoid extends Monoid[(Double, Int)] {
  def mappend(a1: (Double, Int), a2: (Double, Int)): (Double, Int) = (((a1._1*a1._2)+(a2._1*a2._2))/(a1._2+a2._2), a1._2+a2._2)
  def mzero : (Double, Int) = (0.0, 0)
}
```

This plays on the idea that we can describe a set of tuples to represent an average. Here the first element is the average, and the second element is the number of elements considered to build the average. As such we can then see that this is indeed a semigroup or a monoid.

This can be used to calculate a list of tuples, where if you just want to calculate the average of a list of numbers, we would just have to emit `(num, 1)` to then calculate the average.

Here are some examples of how this can be used.

```scala
val x: (Double, Int) = avgMonoid.mappend((0,0), (10,2))
println(x)

val xs: List[(Double, Int)] = List((10,2), (5,3), (5,3), (4.2,1))
val xy = xs.fold(avgMonoid.mzero)(avgMonoid.mappend)
println(xy)

val l: List[Double] = List(1,2,3,4,5,6)
val xx: List[(Double, Int)] = l.map(x => (x, 1))
println(xx.fold(avgMonoid.mzero)(avgMonoid.mappend))
```
