## Problem
You want to execute basic SBT commands to accomplish basic build tasks such as compiling, running, and packaging a project.

## Solution
First, cd to the base directory of the project.

There are two basic ways to run sbt commands: directly from the OS command line, or from the interactive sbt shell.  For example, to compile the source files of the project, you can either run the command directly:

```text
$ sbt compile
[info] Set current project to <project-name> (in build file:<path-to-build-file>)
[info] Updating {file:<path-to-build-file>}<project-name>...
[info] Done updating.
[info] Compiling 4 Scala sources to <project-path>/target/scala-2.9.0.1/classes...
[success] Total time: 32 s, completed Jul 30, 2011 2:10:30 PM
$
```

or you can first enter the sbt shell and then execute the compile command:

```text
$ sbt
[info] Set current project to <project-name> (in build file:<path-to-build-file>)
> compile
[info] Updating {file:<path-to-build-file>}<project-name>...
[info] Done updating.
[info] Compiling 4 Scala sources to <project-path>/target/scala-2.9.0.1/classes...
[success] Total time: 33 s, completed Jul 30, 2011 2:12:45 PM
>
```

The first method is better when you want to execute individual commands and move on to other things, and the second is better when you will be running multiple sbt commands.

Along with `compile`, there are a few other sbt commands that you might use on a regular basis:

**Command** | **Description**
------- | -----------
`compile` | Compiles sources 
`run`     | Runs a main class, passing along arguments provided on the command line
`test`    | Executes all tests
`clean`   | Deletes files produced by the build, such as generated sources, compiled classes, and task caches
`package` | Produces the main artifact, such as a binary jar
`console` | Starts the Scala interpreter with the project classes on the classpath
`doc`     | Generates API documentation

## Discussion
This recipe assumes that you have [[installed sbt|Recipe: Install SBT]] on your system, and that the project in question has been set up to use sbt.  This will of course be the case if you downloaded an existing sbt-enabled project, or if you have set up your own project to work with sbt.
