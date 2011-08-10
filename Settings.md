[light definition]: https://github.com/harrah/xsbt/wiki/Basic-Configuration
[full definition]: https://github.com/harrah/xsbt/wiki/Full-Configuration
[ScopedSetting]: http://harrah.github.com/xsbt/latest/api/sbt/ScopedSetting.html
[Scope]: http://harrah.github.com/xsbt/latest/api/sbt/Scope$.html
[SettingKey]: http://harrah.github.com/xsbt/latest/api/sbt/SettingKey.html
[InputKey]: http://harrah.github.com/xsbt/latest/api/sbt/InputKey.html
[TaskKey]: http://harrah.github.com/xsbt/latest/api/sbt/TaskKey.html

## Introduction

(This page is still a work in progress.  It is intended to become the starting point for learning configuration in 0.10.)

A build definition is written in Scala.
There are two types of definitions: light and full.
A [light definition] is a quick way of configuring a build, consisting of a list of expressions describing project settings.
A [full definition] is made up of one or more Scala source files that describe relationships between projects and introduce new configurations and settings.
This page introduces the `Setting` type, which is used by light and full definitions for general configuration.

### Introductory Examples

Basic examples of each type of definition are shown below for the purpose of getting an idea of what they look like, not for full comprehension of details, which are described at [light definition] and [full definition].

`<base>/build.sbt` (light)
```scala
name := "My Project"

libraryDependencies += "junit" % "junit" % "4.8" % "test"
```

`<base>/project/Build.scala` (full)
```scala
import sbt._
import Keys._

object MyBuild extends Build
{
   lazy val root = Project("root", file(".")) dependsOn(sub)
   lazy val sub = Project("sub", file("sub")) settings(
      name := "My Project",
      libraryDependencies += "junit" % "junit" % "4.8" % "test"
   )
}
```

## Important Settings Background

The fundamental type of a configurable in sbt 0.10 is a `Setting[T]`.
Each line in the `build.sbt` example above is of this type.
The arguments to the `settings` method in the `Build.scala` example are of type `Setting[T]`.
Specifically, the `name` setting has type `Setting[String]` and the `libraryDependencies` setting has type `Setting[Seq[ModuleID]]`, where `ModuleID` represents a dependency.

Throughout the documentation, many examples show a setting, such as:

```scala
libraryDependencies += "junit" % "junit" % "4.8" % "test"
```

This setting either goes in a [light definition] `(build.sbt)` as is or in the `settings` of a `Project` instance in a [full definition] `(Build.scala)` as shown in the example.
This is an important point to understanding the context of examples in the documentation.
(That is, you are a bit more prepared to copy and paste examples now.)

A `Setting[T]` describes how to initialize a setting of type T.  The settings shown in the examples are expressions, not statements.  In particular, there is no hidden mutable map that is being modified.  Each `Setting[T]` is a value that describes an update to a map.  The actual map is rarely directly referenced by user code.  It is not the final map that is usually important, but the operations on the map.

To emphasize this, the setting in the following `Build.scala` fragment *is ignored* because it is a value that need to be included in the `settings` of a `Project`.
(Unfortunately, Scala will discard non-Unit values to get Unit, which is why there is no compile error.)

```scala
object Bad extends Build {
  libraryDependencies += "junit" % "junit" % "4.8" % "test"
}
```

```scala
object Good extends Build
{
  lazy val root = Project("root", file(".")) settings(
    libraryDependencies += "junit" % "junit" % "4.8" % "test"
  )
}
```

## Declaring a Setting

There is fundamentally one type of initialization, represented by the `<<=` method.
The other initialization methods `:=`, `+=`, `++=`, `<+=`, `<++=`, and `~=` can be defined in terms of it.

The motiviation behind the method names is:

* All method end with `=` to obtain the lowest possible infix precedence.
* A method starting with `<` indicates that the initialization uses other settings.
* A single `+` means a single value is expected and will be appended to the current sequence.
* `++` means a `Seq[T]` is expected.  The sequence will be appended to the current sequence.

The following sections include descriptions and examples of each initialization method.
The descriptions use "will initialize" or "will append" to emphasize that they construct a value describing an update and do not mutate anything.
Each setting may be directly included in a light configuration (build.sbt), appropriately separated by blank lines.
For a full configuration (Build.scala), the setting must go in a settings Seq as described in the previous section.
Information about the types of the left and right hand sides of the methods follows this section.

### :=

`:=` is used to define a setting that overwrites any previous value without referring to other settings.
For example, the following defines a setting that will set _name_ to "My Project" regardless of whether _name_ has already been initialized.
```scala
name := "My Project"
```

No other settings are used.  The value assigned is just a constant.

### += and ++=

`+=` is used to define a setting that will append a single value to the current sequence without referring to other settings.
For example, the following defines a setting that will append a JUnit dependency to _libraryDependencies_.
No other settings are referenced.

```scala
libraryDependencies += "junit" % "junit" % "4.8" % "test"
```

The related method `++=` appends a sequence to the current sequence, also without using other settings.
For example, the following defines a setting that will add dependencies on ScalaCheck and specs to the current list of dependencies.
Because it will append a `Seq`, it uses ++= instead of +=.

```scala
libraryDependencies ++= Seq(
   "org.scala-tools.testing" %% "scalacheck" % "1.9" % "test",
   "org.scala-tools.testing" %% "specs" % "1.6.8" % "test"
	)
)
```
### ~=

`~=` is used to transform the current value of a setting.
For example, the following defines a setting that will remove `-Y` compiler options from the current list of compiler options.

```scala
scalacOptions in Compile ~= { (options: Seq[String]) =>
   options filterNot ( _ startsWith "-Y" )
}
```

The earlier declaration of JUnit as a library dependency using `+=` could also be written as:

```scala
libraryDependencies ~= { (deps: Seq[ModuleID]) =>
  deps :+ ("junit" % "junit" % "4.8" % "test")
}
```

### <<=

The most general method is <<=.
All other methods can be implemented in terms of <<=.
<<= defines a setting using other settings, possibly including the previous value of the setting being defined.
For example, declaring JUnit as a dependency using <<= would look like:

```scala
libraryDependencies <<= libraryDependencies apply { (deps: Seq[ModuleID]) =>
   // Note that :+ is a method on Seq that appends a single value
   deps :+ ("junit" % "junit" % "4.8" % "test")
}
```

This defines a setting that will apply the provided function to the previous value of _libraryDependencies_.
`apply` and `Seq[ModuleID]` are explicit for demonstration only and may be omitted.

### <+= and <++=

The <+= method is a hybrid of the += and <<= methods.
Similarly, <++= is a hybrid of the ++= and <<= methods.
These methods are convenience methods for using other settings to append to the current value of a setting.

For example, the following will add a dependency on the Scala compiler to the current list of dependencies.
Because the _scalaVersion_ setting is used, the method is <+= instead of +=.

```scala
libraryDependencies <+= scalaVersion( "org.scala-lang" % "scala-compiler" % _ )
```

This next example adds a dependency on the Scala compiler to the current list of dependencies.
Because another setting (_scalaVersion_) is used and a Seq is appended, the method is <++=.

```scala
libraryDependencies <++= scalaVersion { sv =>
  ("org.scala-lang" % "scala-compiler" % sv) ::
  ("org.scala-lang" % "scala-swing" % sv) ::
  Nil
}
```

## Setting types

This section provides information about the types of the left and right-hand sides of the initialization methods.  It is currently incomplete.

### Setting Keys (left hand side)

The left hand side of a setting definition is of type [ScopedSetting].
This type has two parts: a key (of type [SettingKey]) and a scope (of type [Scope]).
An unspecified scope is like using `this` to refer to the current context.
The previous examples on this page have not defined an explicit scope. See [[Inspecting Settings]] for details on the axes that make up scopes.

The target (value on the left) of a method like `:=` identifies one of the main constructs in sbt: a setting, a task, or an input task.
It is not an actual setting or task, but a key representing a setting or task.
A setting is a value assigned when a project is loaded.
A task is a unit of work that is run on-demand after a project is loaded and produces a value.
An input task, previously known as a method task in sbt 0.7 and earlier, accepts an input string and produces a task to be run.
(The renaming is because it can accept arbitrary input in 0.10 and not just a space-delimited sequence of arguments like in 0.7.)

A setting key has type [SettingKey], a task key has type [TaskKey], and an input task has type [InputKey].
The remainder of this section only discusses settings.
See [[Tasks]] and [[Input Tasks]] for details on the other types (those pages assume an understanding of this page).

