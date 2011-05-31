[sbt-launch.jar]: http://repo.typesafe.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.9.9/sbt-launch.jar
[mailing list]: http://groups.google.com/group/simple-build-tool/
[xsbt-web-plugin]: https://github.com/siasia/xsbt-web-plugin

The assumption here is that you are familiar with SBT 0.7.x but new to 0.9.x.

0.9.x's many new capabilities can be a bit overwhelming, but this page should help you migrate to 0.9.x with a minimum of fuss.  

## Why move to 0.9.x?

 1. Faster builds (because it is smarter at re-compiling only what it must)
 1. Easier configuration.  For simple projects a single `build.sbt` file in your root directory is easier to create than `project/build/MyProject.scala` was.
 1. No more `lib_managed` directory, reducing disk usage and avoiding backup and version control hassles.
 1. `update` is now much faster and it's invoked automatically by SBT.
 1. Terser output. (Yet you can ask for more details if something goes wrong.)

# Step 1: Install SBT 0.9.x

Builds are announced here: [[https://groups.google.com/forum/#!forum/simple-build-tool]].  As I write the most recent is `0.9.9` and can be downloaded here: [sbt-launch.jar].

You can run 0.9.x the same way that you run 0.7.x, either simply:

    java -jar sbt-launch.jar

Or (as most users do) with a shell script like:

    #!/bin/sh
    if test -f ~/.sbtconfig; then
      . ~/.sbtconfig
    fi
    exec java -Xmx512M ${SBT_OPTS} -jar ~/local/sbt/sbt-launch-0.9.9.jar "$@"

Note, in this script that I have renamed the `sbt-launch.jar` file with the version so that I can continue to use my 0.7.x version for projects that haven't been migrated.  I called this script `sbt9`.

For more details see: [[Setup]].

# Step 2: A simple technique for switching an existing project

Here is a simple technique for switching an existing project to 0.9.x while retaining the ability to switch back again at will.  Some builds, such as those with subprojects, are not suited for this technique, but if you learn how to transition a simple project it will help you do a more complex one next.

## Preserve `project/` for 0.7.x project

Rename your `project/` directory to something like `project-old`.  This will hide it from SBT 0.9.x but keep it in case you want to switch back to 0.7.x.

## Create `build.sbt` for 0.9.x

Create a `build.sbt` file in the root directory of your project. See [[Quick-Configuration-Examples]] for simple examples. [[Basic-Configuration]] has more details, but don't get bogged down in detail you probably won't need at first! If you have a simple project then converting your existing project file to this format is largely a matter of re-writing your dependencies and maven archive declarations in a modified yet familiar syntax.

This `build.sbt` file combines aspects of the old `project/build/ProjectName.scala` and `build.properties` files.  It looks like a property file, yet contains Scala code in a special format.

A `build.properties` file like:

    #Project properties
    #Fri Jan 07 15:34:00 GMT 2011
    project.organization=org.myproject
    project.name=My Project
    sbt.version=0.7.7
    project.version=1.0
    def.scala.version=2.7.7
    build.scala.versions=2.8.1
    project.initialize=false

Now becomes part of your `build.sbt` file with lines like:

```scala
name := "My Project"

version := "1.0"

organization := "org.myproject"

scalaVersion := "2.8.1"
```

Currently, a `project/build.properties` is still needed to explicitly select the sbt version.  For example:

```text
sbt.version=0.9.9
```

## Run SBT 0.9.x

Now launch sbt.  If you're lucky it works and you're done.  For help debugging, see below.

## Switching back to SBT 0.7.x

If you get stuck and want to switch back, you can leave your `build.sbt` file alone. SBT 0.7.x will not understand or notice it. Just rename your 0.9.x `project` directory to something like `project09` and rename the backup of your old project from `project-old` to `project` again.

# Useful hints and tips

## Where has `lib_managed` gone?

By default, SBT 0.9.x loads managed libraries from your ivy cache without copying them to a `lib_managed` directory. This fixes some bugs with the previous solution and keeps your project directory small. If you want to insulate your builds from the ivy cache being cleared, set `retrieveManaged := true` and the dependencies will be copied to `lib_managed` as a build-local cache (while avoiding the issues of `lib_managed` in 0.7.x).

This does mean that existing solutions for sharing libraries with your favoured IDE may not work.  There are 0.9.x plugins for IDEs being developed:

* IntelliJ IDEA: [[https://github.com/teigen/plugins]]
* Netbeans: [[https://github.com/remeniuk/sbt-netbeans-plugin]]
* Eclipse: [[https://github.com/weiglewilczek/sbteclipse]]

## What are the commands I can use?

For a list of commands, run `help`.  For details on a specific command, run `help <command>`.  To view a list of tasks defined on the current project, run `tasks`.  Alternatively, see the [[Running]] page for descriptions of common commands and tasks.

If in doubt start by just trying the old command as it may just work.  The built in TAB completion will also assist you, so you can just press TAB at the beginning of a line and see what you get.

The following commands work pretty much as before out of the box:

    reload
    update
    compile
    test
    test-only
    publish-local
    exit

Note that some commands now require an additional space after the initial symbol:

    `~ test` instead of `~test`
    `+ publish` instead of `~publish`
    `++ 2.8.1` instead of `++2.8.1`

## My last command didn't work but I can't see an explanation. Why?

SBT 0.9.x by default suppresses most stack traces and debugging information.  It has the nice side effect of giving you less noise on screen, but as a newcomer it can leave you lost for explanation.  To see the previous output of a command at a higher verbosity, type `last <task>` where `<task>` is the task that failed or that you want to view detailed output for.  For example, if you find that your `update` fails to load all the dependencies as you expect you can enter:

```text
> last update
```

and it will display the full output from the last run of the `update` command.

## Why have the resolved dependencies in a multi-module project changed?

SBT 0.9.x fixes a flaw in how dependencies get resolved in multi-module projects.  This change ensures that only one version of a library appears on a classpath.

Use `last update` to view the debugging output for the last `update` run.  Use `show update` to view a summary of files comprising managed classpaths.

## My tests all run really fast but some are broken that weren't before!

Be aware that compilation and tests run in parallel by default in SBT 0.9.x. If your test code isn't thread-safe then you may want to change this behaviour by adding one of the following to your `build.sbt`:

```scala
// Execute tests in the current project serially.
// Tests from other projects may still run concurrently.
parallelExecution in Test := false

// Execute everything serially (including compilation and tests)
parallelExecution := false
```

## How do I set log levels?

`warn`, `info`, `debug` and `error` don't work any more.

The new syntax in the SBT 0.9.x shell is:
```text
> set logLevel := Level.Warn
```

Or in your `build.sbt` file write:

```scala
logLevel := Level.Warn
```

## What happened to the web development and Web Start support?

Web application support was split out into a plugin.  See the [xsbt-web-plugin] project.

Web Start support still needs a new home.  Please consider adopting it!  Use the [mailing list] for guidance.

## How are inter-project dependencies different in 0.9.x?

In 0.9.x, there are three types of [[project dependencies|Full Configuration]] (classpath, execution, and configuration) and they are independently defined.  These were combined in a single dependency type in 0.7.x.  A declaration like:

```scala
lazy val a = project("a", "A")
lazy val b = project("b", "B", a)
```

meant that the `B` project had a classpath and execution dependency on `A` and `A` had a configuration dependency on `B`.  Specifically, in 0.7.x:
1. Classpath: Classpaths for `A` were available on the appropriate classpath for `B`.
1. Execution: A task executed on `B` would be executed on `A` first.
1. Configuration: For some settings, if they were not overridden in `A`, they would default to the value provided in `B`.

In 0.9.x, declare the specific type of dependency you want.  See [[Full Configuration]] for details.