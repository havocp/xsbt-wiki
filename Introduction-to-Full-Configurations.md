There are two types of file for configuring a build: a `build.sbt` file in you project root directory, or a `Build.scala` file in your `project/` directory.  The former is often referred to as a "light", "quick" or "basic" configuration and the latter is often referred to as "full" configuration.  This page is about "full" configuration.

# Naming the Scala build file

`Build.scala` is the typical name for this build file but in reality it can be called anything that ends with `.scala` as it is a standard Scala source file and sbt will detect and use it regardless of its name.

# Overview of what goes in the file

The most basic form of this file defines one object which extends `sbt.Build` e.g.:

```scala
import sbt._

object AnyName extends Build {
    
    val anyName = Project("anyname", file("."))
    
    // Declarations go here
}
```

There needs to be at least one `sbt.Project` defined and in this case we are giving it an arbitrary name and saying that it can be found in the root of this project.  In other words we are saying that this is a build file to build the current project.

The declarations define any number of objects which can be used by sbt to determine what to build and how to build it. Typically these declarations will include the declaration of at least one `sbt.Project` object which represents an sbt project.

Most of the time you are not telling sbt what to do, you are simply declaring the dependencies of your project and the particular settings you require. sbt then uses this information to determine how to carry out the tasks you give it when you interact with sbt on the command line. For this reason the order of declarations tends to be unimportant.

When you define something and assign it to a constant the name of the constant is often irrelevant. By defining it and making it part of an object, sbt can then interrogate it and extract the information it requires.  So, for example, the line:

```scala
val apachenet = "commons-net" % "commons-net" % "2.0"
```

defines a dependency and assigns it to the constant `apachenet` but, unless you refer to that constant again in the build file, the name of it is of no significance to sbt.  sbt simply sees that the dependency object exists and uses it when it needs it.

# Combining "light" and "full" configuration files

It is worth noting at this stage that you can have both a `build.sbt` file and a `Build.scala` file for the same project.  If you do this, sbt will combine the configurations.  If there is a clash of declarations then the `build.sbt` file will supersede the `Build.scala` file.

# A simple example comparing a "light" and "full" configuration of the same project

Here is a very simple "light" `build.sbt` file which defines a very simple build project with a single test dependency on "scalacheck":

```scala
name := "My Project"

version := "1.0"

organization := "org.myproject"

scalaVersion := "2.9.0-1"

libraryDependencies += "org.scalatest" % "scalatest_2.9.0" % "1.4.1" % "test"
```

Here is an equivalent "full" `Build.scala` file which defines exactly the same thing:

```scala
import sbt._
import Keys._

object MyProjectBuild extends Build {

  val mySettings = Defaults.defaultSettings ++ Seq(
      name := "My Project",
      version := "1.0",
      organization := "org.myproject",
      scalaVersion := "2.9.0-1",
      libraryDependencies += "org.scalatest" % "scalatest_2.9.0" % "1.4.1" % "test"
  )

  val myProject = Project("MyProject", file("."), settings = mySettings)

}
```

Note that we have to explicity declare the build and project and we have to explicitly append our settings to the default settings.  All of this work is done for us when we use a "light" build file.

To understand what is really going on you may find it helpful to see this `Build.scala` without the imports and associated implicit conversions:

```scala
object MyProjectBuild extends sbt.Build {

  val mySettings = sbt.Defaults.defaultSettings ++ scala.Seq(
      sbt.Keys.name := "My Project",
      sbt.Keys.version := "1.0",
      sbt.Keys.organization := "org.myproject",
      sbt.Keys.scalaVersion := "2.9.0-1",
      sbt.Keys.libraryDependencies += sbt.toGroupID("org.scalatest").%("scalatest_2.9.0").%("1.4.1").%("test")
  )

  val myProject = sbt.Project("MyProject", new java.io.File("."), settings = mySettings)

} 
```


