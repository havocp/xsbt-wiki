A build definition is written in Scala.  There are two types of definitions: light and full.  A light definition is a quick way of configuring a build.  It consists of a list of expressions describing a project.

A full definition is made up of one or more Scala source files that describe relationships between projects, introduce new configurations and settings, and define more complex aspects the build.  The capabilities of a light definition are a proper subset of those of a full definition.

# Light Configuration

## By Example
Create a file with extension `.sbt` in your root project directory (such as `<your-project>/build.sbt`).  This file contains Scala expressions of type `Setting[T]` that are separated by blank lines.  Built-in settings typically have reasonable defaults (an exception is `PublishTo`).  A project typically redefines at least `Name` and `Version` and often `LibraryDependencies`.  All built-in settings are (or will be) listed [in sbt.Keys](https://github.com/harrah/xsbt/blob/0.9/main/Default.scala#L22).

A sample `build.sbt`:
```scala
// Set the project name to the string 'My Project'
Name := "My Project"

// The := method used in Name and Version is one of two fundamental methods.
// The other method is <<=
// All other initialization methods are implemented in terms of these.
Version := "1.0"

// Add a single dependency
LibraryDependencies += "junit" % "junit" % "4.8" % "test"

// Add multiple dependencies
LibraryDependencies ++= Seq(
	"net.databinder" %% "dispatch-google" % "0.7.8",
	"net.databinder" %% "dispatch-meetup" % "0.7.8"	
)

// Exclude backup files by default.  This uses ~=, which accepts a function of
//  type T => T (here T = FileFilter) that is applied to the existing value.
// A similar idea is overriding a member and applying a function to the super value:
//  override lazy val defaultExcludes = f(super.defaultExcludes)
//
DefaultExcludes ~= (filter => filter | "*~")
/*  Some equivalent ways of writing this:
DefaultExcludes ~= (_ | "*~")
DefaultExcludes ~= ( (_: FileFilter) | "*~")
DefaultExcludes ~= ( (filter: FileFilter) => filter | "*~")
*/

// Use the name and version to define the jar name.
JarName <<= (Name, Version) {
	(name: String, version: String) => name + "-" + version + ".jar"
}
/* These are equivalent:
JarName <<= (Name, Version)( (n,v) => n + "-" + v + ".jar")
JarName <<= (Name, Version)( _ + "-" + _ + ".jar")
*/
```

## Notes
* Because everything is parsed as an expression, no semicolons are allowed at the ends of lines.
* All initialization methods end with `=` so that they have the lowest possible precedence.  Except when passing a function literal to `~=`, you do not need to use parentheses for either side of the method.
  Ok:
```scala
LibraryDependencies += "junit" % "junit" % "4.8" % "test"
LibraryDependencies.+=("junit" % "junit" % "4.8" % "test")
DefaultExcludes ~= (_ | "*~")
DefaultExcludes ~= (filter => filter | "*~")
```
  Error:
```scala
DefaultExcludes ~= _ | "*~"

error: missing parameter type for expanded function ((x$1) => DefaultExcludes.$colon$tilde(x$1).$bar("*~"))
DefaultExcludes ~= _ | "*~"
                   ^
error: value | is not a member of sbt.Project.Setting[sbt.FileFilter]
DefaultExcludes ~= _ | "*~"
                ^
```
* A block is an expression, with the last statement in the block being the result.  For example, the following is an expression:
```scala
{
	val x = 3
	def y = 2
	x + y
}
```
An example of using a block to construct a Setting:
```scala
Version := {
	// Define a regular expression to match the current branch
	val current = """\*\s+(\w+)""".r
	// Process the output of 'git branch' to get the current branch
	val branch = "git branch --no-color".lines_!.collect { case current(name) => "-" + name }
	// Append the current branch to the version.
	"1.0" + branch.mkString
}
```
* Remember that blank lines are used to clearly delineate expressions.  This happens before the expression is sent to the Scala compiler, so no blank lines are allowed within a block.

## Comparison to Scala language constructs

A loose comparison to classes in Scala can be made:
```scala
trait Name {
  lazy val name = name0
  def name0 = "default"
}
trait GreatName extends Name {
  override def name0 = "The Great " + super.name0
}
```

 is similar to:
```scala
Name := "default"
Name ~= ("The Great " + _)
```

The `JarName` setting in the By Example section is similar to:
```scala
trait Version {
	lazy val version = version0
	def version0 = "1.0"
}
trait JarName {
	lazy val jarName = jarName0
	def jarName0 = "default.jar"
}
trait MyJarName extends Name with Version with JarName {
  override def jarName0 = name + "-" + version + ".jar"
}
```

## More Information

* A `Setting[T]` describes how to initialize a value of type T.  The expressions shown in the example are expressions, not statements.  In particular, there is no hidden mutable map that is being modified.  Each `Setting[T]` describes an update to a map.  The actual map is rarely directly referenced by user code.  It is not the final map that is important, but the operations on the map.
* There are fundamentally two types of initializations, `:=` and `<<=`.  The methods `+=`, `++=`, and `~=` are defined in terms of these.  `:=` assigns a value, overwriting any existing value.  `<<=` uses existing values to initialize a setting.
  * `Key ~= f` is equivalent to `Key <<= Key(f)`
  * `Key += value` is equivalent to `Key ~= (_ :+ value)` or `Key <<= Key(_ :+ value)`
  * `Key ++= value` is equivalent to `Key ~= (_ ++ value)` or `Key <<= Key(_ ++ value)`
* There can be multiple `.sbt` files per project.  This feature can be used, for example, to put user-specific configurations in a separate file.
* Import clauses are allowed at the beginning of a `.sbt` file.  Since they are clauses, no semicolons are allowed.  The need not be separated by blank lines, but each import must be on one line.  For example,
```scala
import scala.xml.NodeSeq
import math.{abs, pow}
```
 * These imports are defined by default in a `.sbt` file:
```scala
import sbt._
import Process._
import Keys._
```
In addition, the contents of all public `Build` and `Plugin` objects from the full definition are imported.
* sbt uses the blank lines to separate the expressions and then it sends them off to the Scala compiler.  Each expression is parsed, compiled, and loaded independently.  The settings are combined into a Seq[Setting[_]] and passed to the settings engine.  The engine groups the settings by key, preserving order per key though, and then computes the order in which each setting needs to be evaluated.  Cycles and references to uninitialized settings are detected here and dead settings are dropped.  Finally, the settings are transformed into a function that is applied to an initially empty map.
* Because the expressions can be separated before the compiler, sbt only needs to recompile expressions that change.  So, the work to respond to changes is proportional to the number of settings that changed and not the number of settings defined in the build.  If imports change, all expression in the `.sbt` file need to be recompiled.

## Implementation Details (even more information)

Each expression describes an initialization operation.  The simplest operation is context-free assignment using `:=`.  That is, no outside information is used to determine the setting value.  Operations other than `:=` are implemented in terms of `<<=`.  The `<<=` method specifies an operation that requires other settings to be initialized and uses their values to define a new setting.

The target (left side value) of a method like `:=` identifies one of the constructs in sbt: settings, tasks, and input tasks.  It is not an actual setting or task, but a key representing a setting or task.  A setting is a value assigned when a project is loaded.  A task is a unit of work that is run on-demand zero or more times after a project is loaded and also produces a value.  An input task, previously known as a Method Task in 0.7 and earlier, accepts an input string and produces a task to be run.  The renaming is because it can accept arbitrary input in 0.9 and not just a space-delimited sequence of arguments like in 0.7.

A construct (setting, task, or input task) is identified by a scoped key, which is a pair `(Scope, AttributeKey[T])`.  An `AttributeKey` associates a name with a type and is a typesafe key for use in an `AttributeMap`.  Attributes are best illustrated by the `get` and `put` methods on `AttributeMap`:
```scala
def get[T](key: AttributeKey[T]): Option[T]
def put[T](key: AttributeKey[T], value: T): AttributeMap
```

For example, given a value `k: AttributeKey[String]` and a value `m: AttributeMap`, `m.get(k)` has type `Option[String]`.

In sbt, a Scope is mainly defined by a project reference and a configuration (such as 'test' or 'compile').  Project data is stored in a Map[Scope, AttributeMap].  Each Scope identifies a map.  You can sort of compare a Scope to a reference to an object and an AttributeMap to the object's data.

In order to provide appropriate convenience methods for constructing an initialization operation for each construct, an AttributeKey is constructed through either a SettingKey, TaskKey, or InputKey:
```scala
// underlying key: AttributeKey[String]
val Name = SettingKey[String]("name")

// underlying key: AttributeKey[Task[String]]
val Hello = TaskKey[String]("hello")

// underlying key: AttributeKey[InputTask[String]]
val HelloArgs = InputKey[String]("hello-with-args")
```

In the basic expression `Name := "asdf"`, the `:=` method is implicitly available for a `SettingKey` and accepts an argument that conforms to the type parameter of Name, which is String.

The high-level API for constructing settings is defined in [Structure](https://github.com/harrah/xsbt/blob/0.9/main/Structure.scala).  Scopes are defined in [Scope](https://github.com/harrah/xsbt/blob/0.9/main/Scope.scala).  The underlying engine is in [Settings](https://github.com/harrah/xsbt/blob/0.9/util/collection/Settings.scala) and the heterogeneous map is in [Attributes](https://github.com/harrah/xsbt/blob/0.9/util/collection/Attributes.scala).
