
f the work of designing 0.9.x (and soon 1.0.x) you might find some of the explanation of it's many capabilities a bit overwhelming.  This page is an attempt to explain how to migrate to 0.9.x with the minimum of fuss.  The assumption is that you are familiar with SBT 0.7.x but don't know much if anything about 0.9.x.

# Step 1: Install SBT 0.9.x

At the moment the latest builds of SBT 0.9.x are being announced on the `simple-build-tool` discussion forum: [[https://groups.google.com/forum/#!forum/simple-build-tool]].  At the point of writing this page the release is `0.9.8` and can be downloaded here: <http://repo.typesafe.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.9.8/sbt-launch.jar>.

You run this in exactly the same way that you run the current version, in other words:

    java -jar sbt-launch.jar

However, typically this is best turned into a shell script like:

    #!/bin/sh
    if test -f ~/.sbtconfig; then
      . ~/.sbtconfig
    fi
    exec java -Xmx512M ${SBT_OPTS} -jar ~/local/sbt/sbt-launch-0.9.8.jar "$@"

Note, in this script that I have renamed the `sbt-launch.jar` file with the version so that I can continue to use my 0.7.x version for projects that haven't been migrated.  I called this script `sbt9`.

# Step 2: A simple technique for switching an existing project

Here is a simple technique for switching an existing project to 0.9.x whilst retaining the ability to switch back again at will.  This is a really simple approach so there will be plenty of more complex SBT projects with subprojects that will not fit this technique, but if you can work out how to transition a simple project then you are much better placed to start thinking about a more complex one.

## Preserve your 0.7.x project settings

Assuming your project is a simple project conforming to the default directory structure, take the `project/` directory and rename it to something like `project-old`.  This will hide it from SBT 0.9.x whilst backing up the old setting if you want to switch back to 0.7.x.

## Create a new 0.9.x `build.sbt` file

Then create a `build.sbt` file in the root directory of your project.  The simplest and most up to date examples of what this should look like seem to be in [[Quick-Configuration-Examples]].  If you have a simple project then converting your existing project file to this format is largely a matter of re-writing your dependencies and maven archive declarations in the slightly modified but largely the same syntax of the new `build.sbt` file.

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

    name := "My Project"
    version := "1.0"
    organization := "org.myproject"
    scalaVersion := "2.8.1"

## Run SBT 0.8.x

Once you've created your build file, give it a try by launching SBT 0.9.x in the root directory of your project.  In a perfect world this will just run and your done!  More details of how to debug problems are listed in the hints and tips below.

## Switching back to SBT 0.7.x

If you get into a mess and you just need your project to work then don't worry about the `build.sbt` file which SBT 0.7.x will not understand or notice, just rename your new `project` directory (created by SBT 0.9.x) to something like `project09` and rename the backup of your old project form `project-old` to `project` again.  Your done!
