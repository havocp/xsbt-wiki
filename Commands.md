# Commands

# Introduction

There are three main aspects to commands:

* The syntax used by the user to invoke the command, including:
    * Tab completion for the syntax
    * The parser to turn the syntax into an appropriate data structure
* The action to perform using the parsed data structure, including the build `State` before and after the command runs
* Help provided to the user

In sbt, the syntax part, including tab completion, is specified with parser combinators.
If you are familiar with the parser combinators in Scala's standard library, these will look familiar.
The action part is a function `(State, T) => State`, where `T` is the data structure produced by the parser.
`State` provides access to the build state, such as all registered `Command`s, the remaining commands to execute, and all project-related information.
Finally, basic help information may be provided that is used by the `help` command to display command help.

# State and actions

[State](https://github.com/harrah/xsbt/blob/0.9/main/State.scala) is the entry point to all available information in sbt.
Key methods are

* `definedCommands: Seq[Command]`, which returns all registered Command definitions
* `remainingCommands: Seq[String]`, which returns the remaining commands to be run
* `attributes: AttributeMap`, which contains generic data.

An action performs some work and transforms `State`.
The following sections discuss `State => State` transformations.
As mentioned previously, a command typically handles a parsed value as well: `(State, T) => State`.

## Command-related data

A Command can modify the currently registered commands or the commands to be executed.
This is done in the action part by modifying the (immutable) State provided to the command.
A function that registers additional power commands might look like:

```scala
val powerCommands: Seq[Command] = ...

val addPower: State => State =
  (state: State) =>
    state.copy(definedCommands =
      (state.definedCommands ++ powerCommands).distinct
    )
```

This takes the current commands, appends new commands, and drops duplicates.
Alternatively, State has a convenience method for doing the above:

```scala
val addPower2 = (state: State) => state ++ powerCommands
```

Some examples of functions that modify the remaining commands to execute:
```scala
val appendCommand: State => State =
  (state: State) =>
    state.copy(remainingCommands = state.remainingCommands :+ "cleanup")

val insertCommand: State => State =
  (state: State) =>
    state.copy(remainingCommands = "next-command" +: state.remainingCommands)
```

The first adds a command that will run after all currently specified commands run.
The second inserts a command that will run next.
The remaining commands will run after the inserted command completes.

To indicate that a command has failed and execution should not continue, return `state.fail`.

```scala
(state: State) => {
  val success: Boolean = ...
  if(success) state else state.fail
}
```

## Project-related data

Project-related information is stored in `attributes`.
Typically, commands won't access this directly but will instead use a convenience method to extract the most useful information:

```scala
val state: State
val extracted: Extracted = Project.extract(state)
import extracted._
```

[Extracted](https://github.com/harrah/xsbt/blob/0.9/main/Project.scala#L53) provides:

* Access to the current build and project (`currentRef`)
* Access to initialized project setting data (`structure.data`)
* Access to session `Setting`s and the original, permanent settings from `.sbt` and `.scala` files (`session.append` and `session.original`)
* Access to the current [Eval](https://github.com/harrah/xsbt/blob/0.9/compile/Eval.scala) instance for evaluating Scala expressions in the build context.

## Project data
All project data is stored in `structure.data`, which is of type `sbt.Settings[Scope]`.
Typically, one gets information of type `T` in the following way:
```scala
val key: SettingKey[T]
val scope: Scope
val value: Option[T] = key in scope get structure.data
```

Here, a `SettingKey[T]` is typically obtained from [Keys](https://github.com/harrah/xsbt/blob/0.9/main/Keys.scala) and is the same type that is used to define settings in `.sbt` files, for example.
[Scope](https://github.com/harrah/xsbt/blob/0.9/main/Scope.scala) selects the scope the key is obtained for.
There are convenience overloads of `in` that can be used to specify only the required scope axes.
Some examples:

```scala
import Keys._
val extracted: Extracted
import extracted._

// get name of current project
val nameOpt: Option[String] = name in currentRef get structure.data

// get the full classpath in the 'compile' configuration, or Nil if it isn't defined
val cp: Seq[Attributed[File]] = fullClasspath in (currentRef, Compile) get structure.data getOrElse Nil

// get the package options for the `test:package-src` task or Nil if none are defined
val pkgOpts: Seq[PackageOption] = packageOptions in (currentRef, Test, packageSrc) get structure.data getOrElse Nil
```

[BuildStructure](https://github.com/harrah/xsbt/blob/0.9/main/Build.scala#L615) contains information about build and project relationships.
Key members are:

```scala
units: Map[URI, LoadedBuildUnit]
root: URI
```

A `URI` identifies a build and `root` identifies the initial build loaded.
[LoadedBuildUnit](https://github.com/harrah/xsbt/blob/0.9/main/Build.scala#L602) provides information about a single build.
The key members of `LoadedBuildUnit` are:

```scala
// Defines the base directory for the build
localBase: File

// maps the project ID to the Project definition
defined: Map[String, ResolvedProject]
```

[ResolvedProject](https://github.com/harrah/xsbt/blob/0.9/main/Project.scala#L27) has the same information as the `Project` used in a `project/Build.scala` except that [ProjectReferences](https://github.com/harrah/xsbt/blob/0.9/main/Reference.scala) are resolved to `ProjectRef`s.

## Classpaths

Classpaths in sbt 0.9 are of type `Seq[Attributed[File]]`.
This allows tagging arbitrary information to classpath entries.
sbt currently uses this to associate an `Analysis` with an entry.
This is how it manages the information needed for multi-project incremental recompilation.
When you only want the underlying `Seq[File]`, use `Attributed.data`:

```scala
val attributedClasspath: Seq[Attribute[File]] = ...
val classpath: Seq[File] = attributedClasspath.map(_.data)
```

## Running tasks
It can be useful to run a specific project task from a command and get its result.
For example, an IDE-related command might want to get the classpath from a project or a task might analyze the results of a compilation.
The relevant method is `Project.evaluateTask`, which has the following signature:
```scala
def evaluateTask[T](taskKey: ScopedKey[Task[T]], state: State,
  checkCycles: Boolean = false, maxWorkers: Int = ...): Option[Result[T]]
```

For example,
```scala
val eval: State => State = (state: State) => {

	// This selects the main 'compile' task for the current project.
	//   The value produced by 'compile' is of type inc.Analysis,
	//   which contains information about the compiled code.
	val taskKey = Keys.compile in Compile

	// Evaluate the task
	// None if the key is not defined
	// Some(Inc) if the task does not complete successfully (Inc for incomplete)
	// Some(Value(v)) with the resulting value
	val result: Option[Result[inc.Analysis]] = Project.evaluateTask(taskKey, state)
	// handle the result
	result match
	{
		case None => // Key wasn't defined.
		case Some(Inc(inc)) => // error detail- inc is of type Incomplete.  use Incomplete.show(inc) to get an error message
		case Some(Value(v)) => // do something with v: inc.Analysis
	}
}
```

For getting the test classpath of a specific project:
```scala
val projectRef: ProjectRef = ...
val taskKey: Task[Seq[Attributed[File]]] =
  Keys.fullClasspath in (projectRef, Test)
```

# Parsing and tab completion

The second key component to a command is parsing and tab completion.
These are specified using parser combinators or convenience methods for basic cases.
This section describes the parser combinators.

If you are already familiar with Scala's parser combinators, the methods are the same except that their arguments are strict.
There are two additional methods for controlling tab completion that are discussed at the end of the section.

Parser combinators build up a parser from smaller parsers.
A `Parser[T]`, in its most basic usage, is a function `String => Option[T]`.
It accepts a `String` to parse and produces a value wrapped in `Some` if parsing succeeds, or `None` if it fails.
(Error handling and tab completion make this picture more complicated.)

The following examples assume the imports:
```scala
import sbt._
import complete.DefaultParsers._
```

## Basic parsers

The simplest parser combinators match exact inputs:

```scala
// A parser that succeeds if the input is 'x', returning the Char 'x'
//  and failing otherwise
val singleChar: Parser[Char] = 'x'

// A parser that succeeds if the input is "blue", returning the String "blue"
//   and failing otherwise
val litString: Parser[String] = "blue"
```

In these examples, implicit conversions produce a literal `Parser` from a `Char` or `String`.
Other basic parser constructors are the `charClass`, `success` and `failure` methods:

```scala
// A parser that succeeds if the character is a digit, returning the matched Char 
val digit: Parser[Char] = charClass( (c: Char) => c.isDigit)

// A parser that produces the value 3 for an empty input string, fails otherwise
val alwaysSucceed: Parser[Int] = success( 3 )

// Represents failure (always returns None for an input String).
//  If error handling were implemented, an error message would go here.
val alwaysFail: Parser[Nothing] = failure("")
```

## Combining parsers

We build on these basic parsers to construct more interesting parsers.
We can combine parsers in a sequence, choose between parsers, or repeat a parser.

```scala
// A parser that succeeds if the input is "blue" or "green",
//  returning the matched input
val color: Parser[String] = "blue" | "green"

// A parser that matches either "fg" or "bg"
val select: Parser[String] = "fg" | "bg"

// A parser that matches "fg" or "bg", a space, and then the color, returning the matched values.
//   ~ is an alias for Tuple2.
val setColor: Parser[String ~ Char ~ String] =
  select ~ ' ' ~ color
 
// Often, we don't care about the value matched by a parser, such as the space above
//  For this, we can use ~> or <~, which keep the result of
//  the parser on the right or left, respectively
val setColor2: Parser[String ~ String]  =  select ~ (' ' ~> color)

// Match one or more digits, returning a list of the matched characters
val digits: Parser[Seq[Char]]  =  charClass(_.isDigit).+

// Match zero or more digits, returning a list of the matched characters
val digits0: Parser[Seq[Char]]  =  charClass(_.isDigit).*

// Optionally match a digit
val optDigit: Parser[Option[Char]]  =  charClass(_.isDigit).?
```

## Transforming results

A key aspect of parser combinators is transforming results along the way into more useful data structures.
The fundamental methods for this are `map` and `flatMap`.
Here are examples of `map` and some convenience methods implemented on top of `map`.

```scala
// Apply the `digits` parser and apply the provided function to the matched
//   character sequence
val num: Parser[Int] = digits map { (chars: Seq[Char]) => chars.mkString.toInt }

// Match a digit character, returning the matched character or return '0' if the input is not a digit
val digitWithDefault: Parser[Char]  =  charClass(_.isDigit) ?? '0'

// The previous example is equivalent to:
val digitDefault: Parser[Char] =
  charClass(_.isDigit).? map { (d: Option[Char]) => d getOrElse '0' }
  
// Succeed if the input is "blue" and return the value 4
val blue = "blue" ^^^ 4

// The above is equivalent to:
val blueM = "blue" map { (s: String) => 4 }
```

## Controlling tab completion

Most parsers have reasonable default tab completion behavior.
For example, the string and character literal parsers will suggest the underlying literal for an empty input string.
However, it is impractical to determine the valid completions for `charClass`, since it accepts an arbitrary predicate.
The `examples` method defines explicit completions for such a parser:

```scala
val digit = charClass(_.isDigit).examples("0", "1", "2")
```

Tab completion will use the examples as suggestions.
The other method controlling tab completion is `token`.
The main purpose of `token` is to determine the boundaries for suggestions.
For example, if your parser is:

```scala
("fg" | "bg") ~ ' ' ~ ("green" | "blue")
```

then the potential completions on empty input are:
```console
fg green
fg blue
bg green
bg blue
```

Typically, you want to suggest smaller segments or the number of suggestions becomes unmanageable.
A better parser is:

```scala
token( ("fg" | "bg") ~ ' ') ~ token("green" | "blue")
```

Now, the initial suggestions would be (with _ representing a space):
```console
fg_
bg_
```

# Defining a Command

A command combines a function `State => Parser[T]` with an action `(State, T) => State`.
The reason for `State => Parser[T]` and not simply `Parser[T]` is that often the current `State` is used to build the parser.
For example, the currently loaded projects determine valid completions for the `project` command.
Examples for the general and specific cases are shown in the following sections.

See [[https://github.com/harrah/xsbt/blob/0.9/main/Command.scala#L31]] for the source API details for constructing commands.

## General commands

General command construction looks like:

```scala
val action: (State, T) => State = ...
val parser: State => Parser[T] = ...
val command: Command = Command("name")(parser)(action)
```

## No-argument commands

There is a convenience method for constructing commands that do not accept any arguments.

```scala
val action: State => State = ...
val command: Command = Command.command("name")(action)
```

## Single-argument command

There is a convenience method for constructing commands that accept a single argument with arbitrary content.

```scala
// accepts the state and the single argument
val action: (State, String) => State = ...
val command: Command = Command.single("name")(action)
```

## Multi-argument command

There is a convenience method for constructing commands that accept multiple arguments separated by spaces.

```scala
val action: (State, Seq[String]) => State = ...

// <arg> is the suggestion printed for tab completion on an argument
val command: Command = Command.args("name", "<arg>")(action)
```

# Full Example

The following example is a valid `project/Build.scala` that adds commands to a project.
To try it out:

1. Copy the following build definition into `project/Build.scala` for a new project.
2. Run sbt on the project.
3. Try out the `hello`, `hello-all`, `fail-if-true`, `color`, and `print-state` commands.
4. Use tab-completion and the code below as guidance.

```scala
import sbt._
import Keys._

// imports standard command parsing functionality
import complete.DefaultParsers._

object CommandExample extends Build
{
	// Declare a single project, adding several new commands, which are discussed below.
	lazy val projects = Seq(root)
	lazy val root = Project("root", file(".")) settings(
		commands ++= Seq(hello, helloAll, failIfTrue, changeColor, printState)
	)

	// A simple, no-argument command that prints "Hi",
	//  leaving the current state unchanged.
	def hello = Command.command("hello") { state =>
		println("Hi!")
		state
	}


	// A simple, multiple-argument command that prints "Hi" followed by the arguments.
	//   Again, it leaves the current state unchanged.
	def helloAll = Command.args("hello-all", "<name>") { (state, args) =>
		println("Hi " + args.mkString(" "))
		state
	}


	// A command that demonstrates failing or succeeding based on the input
	def failIfTrue = Command.single("fail-if-true") {
		case (state, "true") => state.fail
		case (state, _) => state
	}


	// Demonstration of a custom parser.
	// The command changes the foreground or background terminal color
	//  according to the input.
	lazy val change = Space ~> (reset | setColor)
	lazy val reset = token("reset" ^^^ "\\033[0m")
	lazy val color = token( Space ~> ("blue" ^^^ "4" | "green" ^^^ "2") )
	lazy val select = token( "fg" ^^^ "3" | "bg" ^^^ "4" )
	lazy val setColor = (select ~ color) map { case (g, c) => "\\033[" + g + c + "m" }

	def changeColor = Command("color")(_ => change) { (state, ansicode) =>
		print(ansicode)
		state
	}


	// A command that demonstrates getting information out of State.
	def printState = Command.command("print-state") { state =>
		import state._
		println(definedCommands.size + " registered commands")
		println("commands to run: " + show(commands))
		println()

		println("original arguments: " + show(configuration.arguments))
		println("base directory: " + configuration.baseDirectory)
		println()

		println("sbt version: " + configuration.provider.id.version)
		println("Scala version (for sbt): " + configuration.provider.scalaProvider.version)
		println()

		val extracted = Project.extract(state)
		import extracted._
		println("Current build: " + currentRef.build)
		println("Current project: " + currentRef.project)
		println("Original setting count: " + session.original.size)
		println("Session setting count: " + session.append.size)

		state
	}

	def show[T](s: Seq[T]) =
		s.map("'" + _ + "'").mkString("[", ", ", "]")
}
```
