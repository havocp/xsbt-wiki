[IvyConsole]: http://harrah.github.com/xsbt/latest/sxr/IvyConsole.scala.html
[conscript]: https://github.com/n8han/conscript
[setup script]: https://github.com/paulp/xsbtscript

# Scripts, REPL, and Dependencies

sbt has two alternative entry points that may be used to:

* Compile and execute a Scala script containing dependency declarations or other sbt settings
* Start up the Scala REPL, defining the dependencies that should be on the classpath

These entry points should be considered experimental.  A notable disadvantage of these approaches is the startup time involved.

# Setup

To set up these entry points, you can either use [conscript] or manually construct the startup scripts.
In addition, there is a [setup script] for the script mode that only requires a JRE installed.

## Setup with Conscript

Install [conscript].

```
cs harrah/xsbt --branch v0.11.0
```

This will create two scripts: `screpl` and `scalas`.

## Manual Setup

Duplicate your standard `sbt` script, which was set up according to [[Setup]], as `scalas` and `screpl` (or whatever names you like).

`scalas` is the script runner and its command line should look like:

```scala
java -Dsbt.main.class=sbt.ScriptMain -Dsbt.boot.directory=/home/user/.sbt/boot -jar sbt-launch.jar "$@"
```

For the REPL runner, use sbt.ConsoleMain for the main class:

```scala
java -Dsbt.main.class=sbt.ConsoleMain -Dsbt.boot.directory=/home/user/.sbt/boot -jar sbt-launch.jar "$@"
```

In each case, `/home/user/.sbt/boot` should be replaced with wherever you want sbt's boot directory to be.

# Usage

## sbt Script runner

The script runner can run a standard Scala script, but with the additional ability to configure sbt.
sbt settings may be embedded in the script in a comment block that opens with `/***`.

### Example

Copy the following script and make it executable.
You may need to adjust the first line depending on your script name and operating system.
When run, the example should retrieve Scala, the required dependencies, compile the script, and run it directly.
For example, if you name it `dispatch_example.scala`, you would do on Unix:

```
chmod u+x dispatch_example.scala
./dispatch_example.scala
```

```scala
#!/usr/bin/env scalas
!#

/***
scalaVersion := "2.9.0-1"

libraryDependencies ++= Seq(
  "net.databinder" %% "dispatch-twitter" % "0.8.3",
  "net.databinder" %% "dispatch-http" % "0.8.3"
)
*/

import dispatch.{ json, Http, Request }
import dispatch.twitter.Search
import json.{ Js, JsObject }

def process(param: JsObject) = {
  val Search.text(txt)        = param
  val Search.from_user(usr)   = param
  val Search.created_at(time) = param

  "(" + time + ")" + usr + ": " + txt
}

Http.x((Search("#scala") lang "en") ~> (_ map process foreach println))
```

## sbt REPL with dependencies

The arguments to the REPL mode configure the dependencies to use when starting up the REPL.
An argument may be either a jar to include on the classpath, a dependency definition to retrieve and put on the classpath, or a resolver to use when retrieving dependencies.

A dependency definition looks like:

```
organization%module%revision
```

Or, for a cross-built dependency:

```
organization%%module%revision
```

A repository argument looks like:

```
"id at url"
```

This syntax was a quick hack.  Feel free to improve it.  The relevant class is [IvyConsole].