[sbt-launch.jar]: http://typesafe.artifactoryonline.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.11.0/sbt-launch.jar

# Setup

This page is the first in a series of Getting Started pages that will introduce you to sbt.

Please go in order! Later pages assume you're familiar with earlier concepts.

# Overview

To create an sbt project, you'll need to take these steps:

 - Install sbt and create a script to launch it.
 - Setup a simple [[Hello World]] project
    - Create a project directory with source files in it.
    - Create your build definition.
 - Move on to [[Running]] to learn how to run sbt.
 - Then move on to [[Basic Build Definition]] to learn more about build definitions.

# Installing sbt

You need two files; [sbt-launch.jar] and a script to run it.

## Unix

Download [sbt-launch.jar] and place it in `~/bin`.

Create a script to run the jar, by placing this in a file called `sbt` in your `~/bin` directory:

```text
java -Dsbt.boot.directory=$HOME/.sbt/boot/ -Xmx512M -jar `dirname $0`/sbt-launch.jar "$@"
```

Make the script executable:

```text
$ chmod u+x ~/bin/sbt
```

## Windows

Create a batch file `sbt.bat`:

```text
set SCRIPT_DIR=%~dp0
java -Xmx512M -jar "%SCRIPT_DIR%sbt-launch.jar" %*
```

and put [sbt-launch.jar] in the same directory as the batch file.  Put `sbt.bat` on your path so that you can launch `sbt` in any directory by typing `sbt` at the command prompt.

## Tips and Notes

If you have any trouble running `sbt`, see [[Setup Notes]] on terminal encodings, HTTP proxies, and JVM options.

## Next

Move on to [[create a simple project|Hello World]].
