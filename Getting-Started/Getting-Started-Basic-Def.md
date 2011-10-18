[Keys]: http://harrah.github.com/xsbt/latest/sxr/Keys.scala.html

# `.sbt` Build Definition

[[Previous|Getting Started Running]] _Getting Started Guide page 6 of 14._ [[Next|Getting Started Scopes]]

This page describes sbt build definitions, including some "theory" and the
syntax of `build.sbt`. It assumes you know how to [[use sbt|Getting Started Running]] and
have read the previous pages in the Getting Started Guide.

## `.sbt` vs. `.scala` Definition

An sbt build definition can contain files ending in `.sbt`,
located in the base directory, and files ending in `.scala`,
located in the `project` subdirectory of the base directory.

You can use either one exclusively, or use both. A good approach
is to use `.sbt` files for most purposes, and use `.scala` files
only to contain what can't be done in `.sbt`:

 - to customize sbt (add new settings or tasks)
 - to define nested sub-projects

This page discusses `.sbt` files.  See
[[.scala build definition|Getting Started Full Def]] (later in
Getting Started) for more on `.scala` files and how they relate to
`.sbt` files.

## What is a build definition?

** PLEASE READ THIS SECTION **

After examining a project and processing any build definition files, sbt
will end up with an immutable map (set of key-value pairs) describing the
build.

For example, one key is `name` and it maps to a string value, the name of
your project.

_Build definition files do not affect sbt's map directly._

Instead, the build definition creates a huge list of objects with type
`Setting[T]` where `T` is the type of the value in the map. (Scala's
`Setting[T]` is like `Setting<T>` in Java.) A `Setting` describes a
_transformation to the map_, such as adding a new key-value pair or
appending to an existing value.  (In the spirit of functional programming, a
transformation returns a new map, it does not update the old map in-place.)

In `build.sbt`, you might create a `Setting[String]` for the name of your
project like this:

```scala
name := "hello"
```

This `Setting[String]` transforms the map by adding (or replacing) the
`name` key, giving it the value `"hello"`. The transformed map becomes sbt's
new map.

To create its map, sbt first sorts the list of settings so that
all changes to the same key are made together, and values that depend on
other keys are processed after the keys they depend on. Then sbt walks over
the sorted list of `Setting` and applies each one to the map in turn.

Summary: _A build definition defines a list of `Setting[T]`, where a
`Setting[T]` is a transformation affecting sbt's map of key-value pairs and
`T` is the type of each value_.

## How `build.sbt` defines settings

Here's an example:

```scala
name := "hello"

version := "1.0"

scalaVersion := "2.9.1"
```

A `build.sbt` file is a list of `Setting`, separated by blank lines. Each
`Setting` is defined with a Scala expression.

The expressions in `build.sbt` are independent of one another, and
they are expressions, rather than complete Scala statements. An
implication of this is that you can't define a top-level `val`,
`object`, class, or method in `build.sbt`.

On the left, `name`, `version`, and `scalaVersion` are _keys_. A
key is an instance of `SettingKey[T]`, `TaskKey[T]`, or
`InputKey[T]` where `T` is the expected value type. The kinds of
key are explained more below.

Keys have a method called `:=`, which returns a `Setting[T]`. You could
use a Java-like syntax to call the method:

```scala
name.:=("hello")
```

But Scala allows `name := "hello"` instead (in Scala, any method can use either syntax).

The `:=` method on key `name` returns a `Setting`, specifically a
`Setting[String]`. `String` also appears in the type of `name` itself, which
is `SettingKey[String]`. In this case, the returned `Setting[String]` is
a transformation to add or replace the `name` key in sbt's map, giving it
the value `"hello"`.

If you use the wrong value type, the build definition will not compile:

```scala
name := 42  // will not compile
```

## Keys are defined in the Keys object

The built-in keys are just fields in an object called [Keys]. A
`build.sbt` implicitly has an `import sbt.Keys._`, so
`sbt.Keys.name` can be referred to as `name`.

Custom keys may be defined in a
[[.scala file|Getting Started Full Def]] or a [[plugin|Getting Started Using Plugins]].

## Other ways to transform settings

Replacement with `:=` is the simplest transformation, but there are several
others. For example you can append to a list value with `+=`.

The other transformations require an understanding of [[scopes|Getting Started Scopes]], so the
[[next section|Getting Started Scopes]] is about scopes and the
[[section after that|Getting Started More About Settings]] goes into more detail about settings.

## Task Keys

There are three flavors of key:

 - `SettingKey[T]`: a key with a value computed once (the value is
   computed one time when loading the project, and kept around).
 - `TaskKey[T]`: a key with a value that has to be recomputed each time,
   potentially creating side effects.
 - `InputKey[T]`: a task key which has command line arguments as
   input. The Getting Started Guide doesn't cover `InputKey`,
   when you finish this guide, check out [[Input Tasks]] for more.

A `TaskKey[T]` is said to define a _task_. Tasks are operations such as
`compile` or `package`. They may return `Unit` (`Unit` is Scala for `void`),
or they may return a value related to the task, for example `package` is a
`TaskKey[File]` and its value is the jar file it creates.

Each time you start a task execution, for example by typing `compile` at the
interactive sbt prompt, sbt will re-run any tasks involved exactly once.

sbt's map describing the project can cache a string value for a setting such
as `name`, but it has to keep around some executable code for a task such as
`compile` -- even if that executable code eventually returns a string, it
has to be re-run every time.

_A given key always refers to either a task or a plain setting._ That is,
"taskiness" (whether to re-run each time) is a property of the key, not the
value.

Using `:=`, you can assign a computation to a task, and that computation will be
re-run each time:

```scala
hello := { println("Hello!") }
```

From a type-system perspective, the `Setting` created from a task key is
slightly different from the one created from a setting key. `taskKey := 42`
results in a `Setting[Task[T]]` while `settingKey := 42` results in a
`Setting[T]`. For most purposes this makes no difference; the task key still
creates a value of type `T` when the task executes.

The `T` vs. `Task[T]` type difference has this implication: a setting key
can't depend on a task key, because a setting key is cached, and not
re-run. More on this in [[more about settings|Getting Started More About Settings]], coming up soon.

## Keys in sbt interactive mode

In sbt's interactive mode, you can type the name of any task to
execute that task. This is why typing `compile` runs the compile
task. `compile` is a task key.

If you type the name of a setting key rather than a task key, the
value of the setting key will be displayed. Typing a task key name
executes the task but doesn't display the resulting value; to see
a task's result, use `show <task name>` rather than plain `<task
name>`.

In build definition files, keys are named with `camelCase` following Scala
convention, but the sbt command line uses `hyphen-separated-words`
instead. The hyphen-separated string used in sbt comes from the definition of the key (see
[Keys]). For example, in `Keys.scala`, there's this key:

```scala
val scalacOptions = TaskKey[Seq[String]]("scalac-options", "Options for the Scala compiler.")
```

In sbt you type `scalac-options` but in a build definition file you use `scalacOptions`.

To learn more about any key, type `inspect <keyname>` at the sbt interactive
prompt. Some of the information `inspect` displays won't make sense yet, but
at the top it shows you the setting's value type and a brief description of
the setting.

## Imports in `build.sbt`

You can place import statements at the top of `build.sbt`; they need not be
separated by blank lines.

There are some implied default imports, as follows:

```scala
import sbt._
import Process._
import Keys._
```

(In addition, if you have [[.scala files|Getting Started Full Def]],
the contents of any `Build` or `Plugin` objects in those files will be
imported. More on that when we get to
[[.scala build definitions|Getting Started Full Def]].)

## Adding library dependencies

To depend on third-party libraries, there are two options. The
first is to drop jars in `lib/` (unmanaged dependencies) and the
other is to add managed dependencies, which will look like this in
`build.sbt`:

```scala
libraryDependencies += "org.apache.derby" % "derby" % "10.4.1.3"
```

This is how you add a managed dependency on the Apache Derby
library, version 10.4.1.3.

The `libraryDependencies` key involves two complexities: `+=`
rather than `:=`, and the `%` method. `+=` appends to the key's
old value rather than replacing it, this is explained in
[[more about settings|Getting Started More About Settings]]. The
`%` method is used to construct an Ivy module ID from strings,
explained in
[[library dependencies|Getting Started Library Dependencies]].

We'll skip over the details of library dependencies until later in
the Getting Started Guide. There's a
[[whole page|Getting Started Library Dependencies]] covering it
later on.

## Next

Move on to [[learn about scopes|Getting Started Scopes]].
