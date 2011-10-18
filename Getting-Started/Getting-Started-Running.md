# Running

[[Previous|Getting Started Directories]] _Getting Started Guide page 5 of 14._ [[Next|Getting Started Basic Def]]

This page describes how to use `sbt` once you have set up your project.  It
assumes you've [[installed sbt|Getting Started Setup]] and created a [[Hello, World|Getting Started Hello]] or other project.

## Interactive mode

Run sbt in your project directory with no arguments:

```text
$ sbt
```

Running sbt with no command line arguments starts it in interactive mode.
Interactive mode has a command prompt (with tab completion and
history!).

For example, you could type `compile` at the sbt prompt:

```text
> compile
```

To `compile` again, press up arrow and then enter.

To run your program, type `run`.

To leave interactive mode, type `exit` or use Ctrl+D (Unix) or Ctrl+Z (Windows).

## Batch mode

You can also run sbt in batch mode, specifying a space-separated list of
sbt actions as arguments. For sbt commands that take arguments, pass the command and arguments as one argument to `sbt` by enclosing them in quotes. For example,

```text
$ sbt clean compile "test-only TestA TestB"
```

In this example, `test-only` has arguments, `TestA` and `TestB`. The actions will be
run in sequence (`clean`, `compile`, then `test-only`).

## Continuous build and test

To speed up your edit-compile-test cycle, you can ask sbt to automatically
recompile or run tests whenever you save a source file.

Make an action run when one or more source files change by prefixing the
action with `~`.  For example, in interactive mode try:

```text
> ~ compile
```

Press enter to stop watching for changes.

You can use the `~` prefix with either interactive mode or batch mode.

See [[Triggered Execution]] for more details.

## Common actions

Here are some of the most common sbt commands. For a more complete list, see [[FIXME]].

* `clean`
  Deletes all generated files (the `target` directory).
* `compile`
  Compiles the main sources (in the `src/main/scala` directory).
* `test`
  Compiles and runs all tests.
* `console`
  Starts the Scala interpreter with a classpath including the compiled
  sources and all dependencies. To return to sbt, type `:quit`, Ctrl+D
  (Unix), or Ctrl+Z (Windows).
* `run <argument>*`
  Runs the main class for the project in the same virtual machine as `sbt`.
* `package`
  Creates a jar file containing the files in `src/main/resources` and the classes compiled from `src/main/scala`.
* `help <command>`
  Displays detailed help for the specified command.  If no command is
  provided, displays brief descriptions of all commands.
* `reload`
  Reloads the build definition (`build.sbt`, `project/*.scala`,
  `project/*.sbt` files). Needed if you change the build definition.

## History Commands

Interactive mode remembers history, even if you exit sbt and restart it.
The simplest way to access history is with the up arrow key. The following
commands are also supported:

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

## Next

Move on to [[understanding build.sbt|Getting Started Basic Def]].
