[sbt-launch.jar]: http://repo.typesafe.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.9.8/sbt-launch.jar
[mailing list]: http://groups.google.com/group/simple-build-tool/
[xsbt-web-plugin]: https://github.com/siasia/xsbt-web-plugin

The assumption here is that you are familiar with SBT 0.7.x but new to 0.9.x.

0.9.x's many new capabilities can be a bit overwhelming, but this page should help you migrate to 0.9.x with a minimum of fuss.  

## Why move to 0.9.x at this early stage?

 1. Faster builds (because it is smarter at re-compiling only what it must)
 1. Easier configuration.  For simple projects a single `build.sbt` file in your root directory is easier to create than `project/build/MyProject.scala` was.
 1. No more `lib_managed` directory, reducing disk usage and avoiding backup and version control hassles.

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

Once you've created your build file, give it a try by launching SBT 0.9.x in the root directory of your project.  In a perfect world this will just run and your done!  More details of how to debug problems are listed in the hints and tips below.

## Switching back to SBT 0.7.x

If you get into a mess and you just need your project to work then don't worry about the `build.sbt` file which SBT 0.7.x will not understand or notice, just rename your new `project` directory (created by SBT 0.9.x) to something like `project09` and rename the backup of your old project form `project-old` to `project` again.  You're done!

# Other stuff you might want to know!

## Where has `lib_managed` gone?

One feature of SBT 0.9.x is that it uses managed libraries directly from your ivy cache and not from copies added to your `lib_managed` directory.  This saves your project directory growing outrageously under the weight of dependencies that are already stored multiple times in other places on your machine.

Unfortunately this does mess with your existing solution to sharing your libraries with your favoured IDE.  If you are using IntelliJ IDEA then you might want to try out this SBT 0.9.x plugin that will create your IntelliJ project set up with all your dependencies: [[https://github.com/teigen/plugins]].

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

## My last command didn't work but I can't see an explanation why?

SBT 0.9.x by default suppresses most stack traces and debugging information.  It has the nice side effect of giving you less noise on screen, but as a newcomer it can leave you lost for explanation.  Thankfully there is a simple solution: type `last command` where `command` is the command you last entered that went wrong.  e.g. if you find that your `update` fails to load all the dependencies as you expect you can enter:

```text
> last update
```

and it will display lost of logging detail from the last run of the `update` command.

## My dependencies aren't working any more, why?

SBT 0.9.x fixes a flaw in how dependencies get resolved.  So a list of dependencies that worked fine in 0.7.x might fail under 0.9.x for reasons that are nothing to do with your failure to understand 0.9.x syntax.  The `last update` command is your friend.

## My tests all run really fast but some are broken that weren't before!

Be aware that compiles and tests are carried out in parallel by default in SBT 0.9.x.  If you test code is not thread safe then you may want to force it to run your tests one at a time by adding this line to your `build.sbt`:

```scala
// Execute tests in the current project serially
//   Tests from other projects may still run concurrently.
parallelExecution in Test := false
```

## How do I set log levels? `warn`, `info`, `debug` and `error` don't work any more

The new syntax in SBT 0.9.x is:
```text
> set logLevel := Level.Warn
```

You can also include this in your `build.sbt` file but you just remove the `set` command prefix so it becomes:

```scala
logLevel := Level.Warn
```

## What happened to the web development and webstart support?

Web application support was split out into a plugin.  See the [xsbt-web-plugin] project.

Webstart support still needs a new home.  Please consider adopting it!  Use the [mailing list] for guidance.