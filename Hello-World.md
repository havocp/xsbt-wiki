
# Hello, World

This page assumes you've [[installed sbt|Setup]].

## Create a project directory with source code

A valid sbt project can be a directory containing a single source file. You could create it and run it like this:

```text
  $ mkdir hello
  $ cd hello
  $ echo 'object Hi { def main(args: Array[String]) = println("Hi!") }' > hw.scala
  $ sbt
  ...
  > run
  ...
  Hi!
```

In this case, sbt works purely by convention. You don't need to tell sbt about:

 - Sources in the base directory
 - Sources in `src/main/scala` or `src/main/java`
 - Tests in `src/test/scala` or `src/test/java`
 - Data files in `src/main/resources` or `src/test/resources`
 - jars in `lib`

By default, sbt will build projects with the same version of Scala used to run sbt itself.

When you run `sbt` and there is no build definition, it assumes default settings:

 - main Scala sources go in the base directory or in `src/main/scala`
 - test Scala sources go in `src/test/scala`
 - unmanaged dependencies (jars) go in `lib/`
 - the Scala version used to run sbt is used to build the project

You can run the project with `sbt run` or enter the [Scala REPL](http://www.scala-lang.org/node/2097)
with `sbt console`. `sbt console` sets up your project's classpath so you can
try out live Scala examples based on your project's code.

## Build definition

Most projects will need some manual setup. Basic build definitions are done
in a file called `build.sbt` placed in the project's base directory.

For example, if your project is in the directory `hello`, in `hello/build.sbt` you might write:

```scala
name := "hello"

version := "1.0"

scalaVersion := "2.9.1"
```

In [[Basic Build Definition]] you'll learn more about how to write a `build.sbt` file.

If you plan to package your project in a jar, you will want to set at least
the name and version in a `build.sbt`.

## Setting the sbt version

You can force a particular version of sbt by creating a file `hello/project/build.properties`.
In this file, write:

```text
sbt.version=0.11.0
```

From 0.10 onwards, sbt is 99% compatible from release to release. Still,
setting the sbt version in `project/build.properties` avoids any potential
confusion.

# Next

Learn about the [[file and directory layout|Directory Structure]] of an sbt project.

