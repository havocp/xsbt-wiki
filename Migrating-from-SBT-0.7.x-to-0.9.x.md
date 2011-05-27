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

At the moment the latest builds of SBT 0.9.x are being announced on the `simple-build-tool` discussion forum: [[https://groups.google.com/forum/#!forum/simple-build-tool]].  At the point of writing this page the release is `0.9.8` and can be downloaded here: [sbt-launch.jar].

You run this in exactly the same way that you run the current version, in other words:

    java -jar sbt-launch.jar

However, typically this is best turned into a shell script like:

    #!/bin/sh
    if test -f ~/.sbtconfig; then
      . ~/.sbtconfig
    fi
    exec java -Xmx512M ${SBT_OPTS} -jar ~/local/sbt/sbt-launch-0.9.8.jar "$@"

Note, in this script that I have renamed the `sbt-launch.jar` file with the version so that I can continue to use my 0.7.x version for projects that haven't been migrated.  I called this script `sbt9`.

There's a more detailed explanation of all this here: [[Setup]].

# Step 2: A simple technique for switching an existing project

Here is a simple technique for switching an existing project to 0.9.x whilst retaining the ability to switch back again at will.  This is a really simple approach so there will be plenty of more complex SBT projects with subprojects that will not fit this technique, but if you can work out how to transition a simple project then you are much better placed to start thinking about a more complex one.

## Preserve your 0.7.x project settings

Assuming your project is a simple project conforming to the default directory structure, take the `project/` directory and rename it to something like `project-old`.  This will hide it from SBT 0.9.x whilst backing up the old setting if you want to switch back to 0.7.x.

## Create a new 0.9.x `build.sbt` file

Then create a `build.sbt` file in the root directory of your project.  The simplest and most up to date examples of what this should look like seem to be in [[Quick-Configuration-Examples]] and there is a more detailed explanation in [[Basic-Configuration]] but don't get overwhelmed with the detail which you probably don't need at first! If you have a simple project then converting your existing project file to this format is largely a matter of re-writing your dependencies and maven archive declarations in the slightly modified but largely the same syntax of the new `build.sbt` file.

This `build.sbt` file is a bit like the old `project/build/ProjectName.scala` file except that it is more like a properties file and less like Scala.  It also contains some details which used to be set from the command line in SBT and used to be stored in the `project/build.properties` file.  So a `build.properties` file that looked like:

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