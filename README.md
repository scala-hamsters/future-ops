# FutureOps

This is a port of Hamsters FutureOps to Scala 2.13. 

## Goal

Monad stacks are not easy to compose (e.g. in a for comprehension), and if you don't want to use Monad transformers you can use this Future additional methods.
 
## FutureOps.fromEither

```scala
import io.github.hamsters.FutureOps

FutureOps.fromEither(Right("a")) //Future("a")
FutureOps.fromEither(Left(BoomError)) //Future(BoomError)
```

## squash Future[Either[Throwable, A]] and Future[Try[A]]

You can use `squash` on a `Future[Either[Throwable, A]]` to get a `Future[A]`.

```scala
import FutureOps._

abstract class Error(message: String) extends Exception(message)
case object BoomError extends Error("Boom")

val fea: Future[Either[Error, String]] = Future(Right("a"))
val feb: Future[Either[Error, String]] = Future(Left(BoomError))

fea.squash //Future("a")
feb.squash //Future(BoomError)
```

You can also `squash` on a `Future[Try[A]]` to get a `Future[A]` in much the same way:

```scala
import FutureOps._

val fta: Future[Try[String]] = Future(Success("a"))
val ftb: Future[Try[String]] = Future(Failure(new Exception("Boom")))

fta.squash //Future("a")
ftb.squash //Future(Exception("Boom"))
```

It can also be useful to compose several `Future[Either[Throwable, _]]` without monad transformers :

```scala
def fea: Future[Either[Throwable, Int]] = Future(Right(1))
def feb(a: Int): Future[Either[Throwable, Int]] = Future(Right(a + 2))

val composedAB: Future[Int] = for {
  a <- fea.squash
  ab <- feb(a).squash
} yield ab

composedAB // Future("ab")

val error: Either[Throwable, Int] = Left(BoomError)
val composedABWithError: Future[Int] = for {
  a <- Future.successful(error).squash
  ab <- feb(a).squash
} yield ab

composedABWithError //Future(Failure(BoomError))

```

Composing several `Future[Try[_]]`s without monad transformers is also possible:

```scala
def fta: Future[Try[Int]] = Future(Success(1))
def ftb(a: Int): Future[Try[Int]] = Future(Success(a + 2))

val composedAB: Future[Int] = for {
  a <- fta.squash
  ab <- ftb(a).squash
} yield ab

composedAB // Future("ab")
```

## Options

Same operations can be used with options : `FutureOps.fromOption` and `squash` on `Future[Option[A]]`.
For empty options, an `EmptyValueError` will be raised.

# Install

Add this to you build.sbt : 

```scala
resolvers += Resolver.bintrayRepo("loicdescotte", "Hamsters") 
libraryDependencies += "io.github.scala-hamsters" %% "future-ops" % "1.0.0"
```

