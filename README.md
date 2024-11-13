# slickless

Shapeless HList support for Slick. 

This is a fork of [slickless](https://github.com/underscoreio/slickless) to upgrade Slick to 3.5

by [Richard Dallaway][d6y],
[Miles Sabin][milessabin],
and [Dave Gurnell][davegurnell].

Copyright 2015-2019 [Underscore Consulting LLP][underscore].
Licensed [Apache 2][license].

## Versions

| Scala | Slick | Slickless                                                                                                                                                                                  |
|-------|-------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2.13  | 3.5   | 0.4.0                                                                                                                                                                                      |

Note, Slick 3.5 dropped support for Scala 2.12.

## Getting Started

Grab the code by adding the following to your `build.sbt`:

~~~
libraryDependencies ++= Seq(
  "com.typesafe.slick" %% "slick"     % "3.5.1",
  "com.chuusai"        %% "shapeless" % "2.3.12",
  "com.mosaicpower"    %% "slickless" % "<<VERSION>>"
)
~~~


## Synopsis

Import Slick, shapeless, and slickless,
and you should be able to define `Tables` on any shapeless `HList` type:

~~~ scala
import slick.jdbc.H2Profile.api._
import shapeless.{ HList, ::, HNil }
import slickless._

class Users(tag: Tag) extends Table[Long :: String :: HNil](tag, "users") {
  def id    = column[Long]( "id", O.PrimaryKey, O.AutoInc )
  def email = column[String]("email")

  def * = id :: email :: HNil
}

lazy val users = TableQuery[Users]
~~~

If you want to map your `HList` to a case class
(i.e. you have a case class that has more than
22 fields and you are using slickless to bypass this limit),
you can do the following

~~~ scala
import slick.jdbc.H2Profile.api._
import shapeless.{ HList, ::, HNil, Generic }
import slickless._

case class User(id: Long, email: String)

class Users(tag: Tag) extends Table[User](tag, "users") {
  def id    = column[Long]( "id", O.PrimaryKey, O.AutoInc )
  def email = column[String]("email")

  def * = (id :: email :: HNil).mappedWith(Generic[User])
}

lazy val users = TableQuery[Users]
~~~

## Notes

### Compile time

Due to this [issue](https://github.com/milessabin/shapeless/issues/619),
if you accidentally make a mapping which is incorrect,
the Scala compiler can take a huge amount of time to report an error.
If your slickless project is taking an insanely long amount of time to compile
(more than a couple of minutes),
try to make sure you have the mapping correct before using `<>`.

### Publishing

We use the [sbt-pgp plugin](http://www.scala-sbt.org/sbt-pgp/usage.html) and
the [sbt-sonatype plugin](https://github.com/xerial/sbt-sonatype)
to publish to [Maven Central](https://issues.sonatype.org/browse/OSSRH-24293).

Publish sequence could be:

```
sbt> set pgpPassphrase := Some(Array('s','e','c','r','3','t'))
sbt> set publishTo := sonatypePublishTo.value
sbt> +publishSigned
sbt> sonatypeRelease
```

[d6y]: https://github.com/d6y
[milessabin]: https://github.com/milessabin
[davegurnell]: https://github.com/davegurnell

[underscore]: http://underscore.io
[license]: http://www.apache.org/licenses/LICENSE-2.0
