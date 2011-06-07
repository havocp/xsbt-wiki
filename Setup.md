[sbt-launch.jar]: http://typesafe.artifactoryonline.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.10.0/sbt-launch.jar
[ScalaCheck]: http://code.google.com/p/scalacheck/
[specs]: http://code.google.com/p/specs/
[ScalaTest]: http://www.artima.com/scalatest/
[Maven]: http://maven.apache.org/

# Setup (In Progress)

# Introduction

This page describes how to set up your project for use with sbt.  The basic steps are:

* Create a script to launch sbt.
* Create your sources and put jars in `lib/`.
* Create your build definition(s).
* Read [[Running]] for basic usage instructions.
* For configuration instructions, see [[Basic Configuration]], [[Full Configuration]], and [[Quick Configuration Examples]]

# Launching Sbt

The easiest way to launch sbt is to create a one-line script.  Download [sbt-launch.jar] if you have not already.

**Note**: do _not_ put `sbt-launch.jar` in your `$SCALA_HOME/lib` directory, your project's `lib` directory, or anywhere it will be put on a classpath.

**Note**: The encoding used by your terminal may differ from Java's default encoding for your platform.  In this case, you will need to add the option `-Dfile.encoding=<encoding>` in the following scripts to set the encoding.

**Note**: Adjust the JVM options as necessary, especially heap and permgen sizes.

A common set of options is:
```text
-Dfile.encoding=UTF8 -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256m
```

**Note**: You can share the `project/boot` directory between all sbt projects on a machine by setting the `sbt.boot.directory` system property to the directory to use.  For example, `-Dsbt.boot.directory=/home/user/.sbt/boot/`.

## Unix
Put the jar in your `~/bin` directory, put the line:
```text
java -Xmx512M -jar `dirname $0`/sbt-launch.jar "$@"
```

in a file called `sbt` in your `~/bin` directory and do:
```text
$ chmod u+x ~/bin/sbt
```

This allows you to launch sbt in any directory by typing `sbt` at the command prompt.

`sbt` will pick up any HTTP proxy settings from the `http.proxy` environment variable. If you are behind a proxy requiring authentication, you must in addition pass flags to set the `http.proxyUser` and `http.proxyPassword` properties:

```text
java -Dhttp.proxyUser=username -Dhttp.proxyPassword=mypassword -Xmx512M -jar `dirname $0`/sbt-launch.jar "$@"
```

## Windows
Create a batch file `sbt.bat`:
```text
set SCRIPT_DIR=%~dp0
java -Xmx512M -jar "%SCRIPT_DIR%sbt-launch.jar" %*
```

and put the jar in the same directory as the batch file.  Put `sbt.bat` on your path so that you can launch `sbt` in any directory by typing `sbt` at the command prompt.

If you are behind a proxy on Windows, add flags to this second line for proxy host, port, and if applicable, username and password:
```text
java -Dhttp.proxyHost=myproxy -Dhttp.proxyPort=8080 -Dhttp.proxyUser=username -Dhttp.proxyPassword=mypassword -Xmx512M -jar "%SCRIPT_DIR%sbt-launch.jar" %*
```

# Run

When you run `sbt` and there is no build definition, it assumes default settings:

 - main Scala sources go in the base directory or in `src/main/scala`
 - test Scala sources go in `src/test/scala`
 - unmanaged dependencies (jars) go in `lib/`
 - the Scala version used to run sbt is used to build the project

With these defaults, you can run `console` to enter the Scala interpreter, or create an application and run it with `run`.  For example:

```text
  $ echo 'object Hi { def main(args: Array[String]) = println("Hi!") }' > hw.scala
  $ sbt
  ...
  > run
  ...
  Hi!
```

# Configure Build

Basic configuration can be done in a `build.sbt` file using Scala code.  For example:

```scala
name := "My Project"

version := "1.0"

scalaVersion := "2.8.1"

libraryDependencies += "junit" % "junit" % "4.8" % "test"
```

See [[Basic Configuration]], [[Full Configuration]], and [[Quick Configuration Examples]] for details.

A specific SBT version can be configured in `project/build.properties`. For example:

```text
sbt.version=0.10.0
```

# Directory Layout

## Sources

Sbt uses the same directory structure as [Maven] for source files by default (all paths are relative to the project directory):

```text
  src/
    main/
      resources/
         <files to include in main jar here>
      scala/
         <main Scala sources>
      java/
         <main Java sources>
    test/
      resources
         <files to include in test jar here>
      scala/
         <test Scala sources>
      java/
         <test Java sources>
```

Other directories in `src/` will be ignored.  Additionally, all hidden directories will be ignored.

All manually managed dependencies (jars) go in the `lib` directory by default.  Hidden directories will be ignored.  If you want to use [ScalaCheck], [specs], or [ScalaTest] for testing, those jars can go in here.  Alternatively, you can configure `sbt` to automatically manage your dependencies (see [[Library Management]]).

## Build Configuration

Build configuration is done in `.sbt` files, such as `build.sbt`, and/or by `.scala` files in the `project` directory:
```text
  build.sbt
  project/
    Build.scala
    boot/
```

The `project/boot` directory is where the versions of Scala and `sbt` used to build the project are downloaded to.

## Products

Generated files (compiled classes, packaged jars, managed files, caches, and documentation) will be written to the `target` directory by default.

## Version Control

As described in the previous sections, `sbt` creates and uses a couple of directories that you will normally want to exclude from version control:
```text
  target/
  project/boot/
```

The above list is suitable as an initial `.gitignore` file for your project.

# Next Step

Read [[Running]] for basic `sbt` usage information.

# Summary

* Create a script to launch sbt.
* Create your sources and put jars in `lib/`.
* Create your build definition(s).
* Read [[Running]] for basic usage instructions.
* For configuration instructions, see [[Basic Configuration]], [[Full Configuration]], and [[Quick Configuration Examples]]
