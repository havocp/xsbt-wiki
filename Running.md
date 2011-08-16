# Running

This page describes how to use `sbt` once you have set up your project (see [[Setup]]).

Run sbt in your project directory.  If you have created a script to start sbt, this should be as simple as:

```
$ sbt
```

This starts sbt in interactive mode.  You are given a prompt at which you type commands.  Tab completion and history are available at this prompt.

**Note**: Since 0.10, sbt launches git to manage assets.  Make sure that `git` is executable from your path.

Alternatively, you can run sbt in batch mode.  You specify a space-separated list of actions as arguments.  For commands that take arguments, pass the command and arguments as one argument to `sbt` by enclosing them in quotes.  For example,

```text
$ sbt clean compile "test-only TestA TestB"
```

You can make an action run when one or more source files change by prefixing the action with `~`.  For example:

```text
> ~ compile
```

See [[Triggered Execution]] for details.

Temporarily change the logging level and configure how stack traces are displayed by modifying the `log-level` or `trace-level` settings:

```text
> set logLevel := Level.Warn
```

Valid `Level` values are `Debug, Info, Warn, Error`.

You can run an action for multiple versions of Scala by prefixing the action with `+`.  See [[Cross Build]] for details.  You can temporarily switch to another version of Scala using `++ <version>`.  This version does not have to be listed in your build definition, but it does have to be available in a repository.  You can also include the initial command to run after switching to that version.  For example:

```text
> ++ 2.7.7 console-quick
...
Welcome to Scala version 2.7.7.final (Java HotSpot(TM) Server VM, Java 1.6.0).
...
scala>
...
> ++ 2.8.1 console-quick
...
Welcome to Scala version 2.8.1 (Java HotSpot(TM) Server VM, Java 1.6.0).
...
scala>
```

# Tasks

Some tasks produce useful values.  The `toString` representation of these values can be shown using `show <task>` to run the task instead of just `<task>`.
In a multi-project build, execution dependencies and the `aggregate` setting control which tasks from which projects are executed.  See [[Full Configuration]].

## Project-level tasks

* `clean`
  Deletes all generated files (the `target` directory).
* `publish-local`
  Publishes artifacts (such as jars) to the local Ivy repository as described in [[Publishing]].
* `publish`
  Publishes artifacts (such as jars) to the repository defined by the `publish-to` setting, described in [[Publishing]].
* `update`
  Resolves and retrieves external dependencies as described in [[Library Management]].

## Configuration-level tasks

Configuration-level tasks are tasks associated with a configuration.  For example, `compile`, which is equivalent to `compile:compile`, compiles the main source code (the `compile` configuration).  `test:compile` compiles the test source code (test `test` configuration).  Most tasks for the `compile` configuration have an equivalent in the `test` configuration that can be run using a `test:` prefix.

* `compile`
  Compiles the main sources (in the `src/main/scala` directory).  `test:compile` compiles test sources (in the `src/test/scala/` directory).
* `console`
  Starts the Scala interpreter with a classpath including the compiled sources, all jars in the `lib` directory, and managed libraries.  To return to sbt, type `:quit`, Ctrl+D (Unix), or Ctrl+Z (Windows).  Similarly, `test:console` starts the interpreter with the test classes and classpath.
* `console-quick`
  Starts the Scala interpreter with the project's compile-time dependencies on the classpath.  `test:console-quick` uses the test dependencies.  This task differs from `console` in that it does not force compilation of the current project's sources.
 * `console-project`
  Enters an interactive session with sbt and the build definition on the classpath.  The build definition and related values are bound to variables and common packages and values are imported.  See [[Console Project]] for more information.
* `doc`
  Generates API documentation for Scala source files in `src/main/scala` using scaladoc.  `test:doc` generates API documentation for source files in `src/test/scala`.
* `package`
  Creates a jar file containing the files in `src/main/resources` and the classes compiled from `src/main/scala`.
  `test:package` creates a jar containing the files in `src/test/resources` and the class compiled from `src/test/scala`.
* `package-doc`
  Creates a jar file containing API documentation generated from Scala source files in `src/main/scala`.
  `test:package-doc` creates a jar containing API documentation for test sources files in `src/test/scala`.
* `package-src`:
  Creates a jar file containing all main source files and resources.  The packaged paths are relative to `src/main/scala` and `src/main/resources`.
  Similarly, `test:package-src` operates on test source files and resources.
* `run <argument>*`
  Runs the main class for the project in the same virtual machine as `sbt`.  The main class is passed the `argument`s provided.  Please see [[Running Project Code]] for details on the use of `System.exit` and multithreading (including GUIs) in code run by this action.
  `test:run` runs a main class in the test code.
* `run-main <main-class> <argument>*`
  Runs the specified main class for the project in the same virtual machine as `sbt`.  The main class is passed the `argument`s provided.  Please see [[Running Project Code]] for details on the use of `System.exit` and multithreading (including GUIs) in code run by this action.
  `test:run-main` runs the specified main class in the test code.
 * `test`
  Runs all tests detected during test compilation.  See [[Testing]] for details.
 * `test-only <test>*`
  Runs the tests provided as arguments.  `*` (will be) interpreted as a wildcard in the test name.  See [[Testing]] for details.
  
## General commands

* `exit` or `quit`
  End the current interactive session or build.  Additionally, `Ctrl+D` (Unix) or `Ctrl+Z` (Windows) will exit the interactive prompt.
* `help <command>`
  Displays detailed help for the specified command.  If no command is provided, displays brief descriptions of all commands.
* `projects`
  List all available projects.  (See [[Full Configuration]] for details on multi-project builds.)
* `project <project-id>`
  Change the current project to the project with ID `<project-id>`.  Further operations will be done in the context of the given project. (See [[Full Configuration]] for details on multiple project builds.)
* `~ <command>`
  Executes the project specified action or method whenever source files change.  See [[Triggered Execution]] for details.
* `< filename`
  Executes the commands in the given file.  Each command should be on its own line.  Empty lines and lines beginning with '#' are ignored
* `+ <command>`
  Executes the project specified action or method for all versions of Scala defined in the `cross-scala-versions` setting.
* `++ <version> <command>`
  Temporarily changes the version of Scala building the project and executes the provided command.  `<command>` is optional.  The specified version of Scala is used until the project is reloaded, settings are modified (such as by the `set` or `session` commands), or `++` is run again.  `<version>` does not need to be listed in the build definition, but it must be available in a repository.
* `; A ; B`
  Execute A and if it succeeds, run B.  Note that the leading semicolon is required.
* `eval <Scala-expression>`
  Evaluates the given Scala expression and returns the result and inferred type.  This can be used to set system properties, as a calculator, fork processes, etc ...
  For example: 

    ```scala
 > eval System.setProperty("demo", "true")
 > eval 1+1
 > eval "ls -l" !
    ```

## Commands for managing the build definition

* `reload [plugins|return]`
  If no argument is specified, reloads the build, recompiling any build or plugin definitions as necessary.
  `reload plugins` changes the current project to the plugin definition (in `project/plugins/`).  This can be useful to directly manipulate the plugin definition.  For example, running `clean` on the plugin definition project will force snapshots to be updated.
  `reload return` changes back to the main project.
* `set <setting-expression>`
  Evaluates and applies the given setting definition.  The setting applies until sbt is restarted, the build is reloaded, or the setting is overridden by another `set` command or removed by the `session` command.  See [[Basic Configuration]] and [[Inspecting Settings]] for details.
* `session <command>`
  Manages session settings defined by the `set` command.  See [[Inspecting Settings]] for details.
* `inspect <setting-key>`
  Displays information about settings, such as the value, description, defining scope, dependencies, delegation chain, and related settings.  See [[Inspecting Settings]] for details.

## History Commands
 * `!`
  Show history command help.
 * `!!`
  Execute the previous command again.
 * `!:`
  Show all previous commands.
 * `!:n`
  Show the last n commands.
 * `!n`
  Execute the command with index `n`, as shown by the `!:` command.
 * `!-n`
  Execute the nth command before this one.
 * `!string`
  Execute the most recent command starting with 'string'
 * `!?string`
  Execute the most recent command containing 'string'

## Command Line Options

System properties can be provided either as JVM options, or as SBT arguments, in both cases as `-Dprop=value`. The following properties influence SBT execution.

<table>
  <thead>
    <tr>
      <td>_Property_</td>
      <td>_Values_</td>
      <td>_Default_</td>
      <td>_Meaning_</td>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>`sbt.log.noformat`</td>
      <td>Boolean</td>
      <td>false</td>
      <td>If true, disable ANSI color codes. Useful on build servers or terminals that don't support color.</td>
    </tr>
    <tr>
      <td>`sbt.global.base`</td>
      <td>Directory</td>
      <td>`~/.sbt`</td>
      <td>The directory containing global settings and plugins</td>
    </tr>
    <tr>
      <td>`sbt.ivy.home`</td>
      <td>Directory</td>
      <td>`~/.ivy2`</td>
      <td>The directory containing the local Ivy repository and artifact cache</td>
    </tr>
  </tbody>
