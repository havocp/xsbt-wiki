[sbt-launch.jar]: http://repo.typesafe.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.9.8/sbt-launch.jar
[mailing list]: http://groups.google.com/group/simple-build-tool/
[xsbt-web-plugin]: https://github.com/siasia/xsbt-web-plugin

The assumption here is that you are familiar with SBT 0.7.x but new to 0.9.x.

0.9.x's many new capabilities can be a bit overwhelming, but this page should help you migrate to 0.9.x with a minimum of fuss.  

## Why move to 0.9.x at this early stage?

 1. Faster builds (because it is smarter at re-compiling only what it must)
 1. Easier configuration.  For simple projects a single `build.sbt` file in your root directory is easier to create than `project/build/MyProject.scala` was.
 1. No more `lib_managed` directory, reducing disk usage and avoiding backup and version control hassles.
 1. `update` is now much faster and it's invoked automatically by SBT.

# Step 1: Install SBT 0.9.x

Builds are announced here: [[https://groups.google.com/forum/#!forum/simple-build-tool]].  As I write the most recent is `0.9.8` and can be downloaded here: [sbt-launch.jar].

You can run 0.9.x the same way that you run 0.7.x, either simply:

    java -jar sbt-launch.jar

Or (as most users do) with a shell script like:

    #!/bin/sh
    if test -f ~/.sbtconfig; then
      . ~/.sbtconfig
    fi
    exec java -Xmx512M ${SBT_OPTS} -jar ~/local/sbt/sbt-launch-0.9.8.jar "$@"

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

## Run SBT 0.9.x

Now launch sbt.  If you're lucky it works and you're done.  For help debugging, see below.

## Switching back to SBT 0.7.x

If you get stuck and want to switch back, you can leave your `build.sbt` file alone. SBT 0.7.x will not understand or notice it. Just rename your 0.9.x `project` directory to something like `project09` and rename the backup of your old project from `project-old` to `project` again.

# Other stuff you might want to know!

## Where has `lib_managed` gone?

By default, SBT 0.9.x loads managed libraries from your ivy cache without copying them to a `lib_managed` directory. This fixes some bugs with the previous solution and keeps your project directory small. If you want to insulate your builds from the ivy cache being cleared, set `retrieveManaged := true` and the dependencies will be copied to `lib_managed` as a build-local cache (avoiding the issues of `lib_managed` in 0.7.x).

Unfortunately this does mean that existing solutions for sharing libraries with your favoured IDE may not work.  If you are using IntelliJ IDEA then you might want to try out this SBT 0.9.x plugin that will create your IntelliJ project set up with all your dependencies: [[https://github.com/teigen/plugins]].

# Useful hints and tips

## What are the commands I can use?

You don't appear to be able to list all the possible commands in the way you could with SBT 0.7.x.  If in doubt start by just trying the old command as it may just work!  The built in TAB completion will also offer you all the command options (as well as lots of stuff that aren't commands!) so you can just press TAB at the beginning of a line and see what you get!

The following commands work pretty much as before out of the box:

    reload
    update
    compile
    test
    test-only
    publish-local
    exit

For detailed help see the `help` command or the [[Running]] page.

## My last command didn't work but I can't see an explanation. Why?

SBT 0.9.x by default suppresses most stack traces and debugging information.  It has the nice side effect of giving you less noise on screen, but as a newcomer it can leave you lost for explanation.  Thankfully there is a simple solution: type `last command` where `command` is the command you last entered that went wrong.  e.g. if you find that your `update` fails to load all the dependencies as you expect you can enter:

```text
> last update
```

and it will display the full output from the last run of the `update` command.

## My dependencies aren't working any more. Why?

SBT 0.9.x fixes a flaw in how dependencies get resolved.  So a list of dependencies that worked fine in 0.7.x might fail under 0.9.x for reasons that are nothing to do with your failure to understand 0.9.x syntax.  The `last update` command is your friend.

## My tests all run really fast but some are broken that weren't before!

Be aware that compilation and tests run in parallel by default in SBT 0.9.x.

If your test code isn't thread-safe then you may want to force it to run your tests one at a time by adding this line to your `build.sbt`:

```scala
// Execute tests in the current project serially
//   Tests from other projects may still run concurrently.
parallelExecution in Test := false
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
