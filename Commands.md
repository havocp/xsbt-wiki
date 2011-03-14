# Commands (Work in progress)

## API
See [[https://github.com/harrah/xsbt/blob/0.9/main/Command.scala#L31]]

## Examples

1. Copy the following build definition into `project/Build.scala` for a new project.
2. Run sbt
3. Try out the `hello`, `hello-all`, `color`, and `print-state` commands.  (Use tab-completion and the code below as guidance until the documentation fills out.)

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
		commands ++= Seq(hello, helloAll, changeColor, printState)
	)

	def hello = Command.command("hello") { state =>
		println("Hi!")
		state
	}
	def helloAll = Command.args("hello-all", "<name>") { (state, args) =>
		println("Hi " + args.mkString(" "))
		state
	}

	lazy val change = Space ~> (reset | setColor)
	lazy val reset = token("reset" ^^^ "\\033[0m")
	lazy val color = token( Space ~> ("blue" ^^^ "4" | "green" ^^^ "2") )
	lazy val select = token( "fg" ^^^ "3" | "bg" ^^^ "4" )
	lazy val setColor = (select ~ color) map { case (g, c) => "\\033[" + g + c + "m" }

	def changeColor = Command("color")(_ => change) { (state, ansicode) =>
		print(ansicode)
		state
	}

	def show[T](s: Seq[T]) =
		s.map("'" + _ + "'").mkString("[", ", ", "]")

	def printState = Command.command("print-state") { state =>
		import state._
		println(processors.size + " registered commands")
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
}
```
