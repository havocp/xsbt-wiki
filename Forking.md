[Fork API]: http://harrah.github.com/xsbt/latest/api/sbt/Fork$.html
[ForkJava]: http://harrah.github.com/xsbt/latest/api/sbt/Fork$.ForkJava.html
[ForkScala]: http://harrah.github.com/xsbt/latest/api/sbt/Fork$.ForkScala.html
[OutputStrategy]: http://harrah.github.com/xsbt/latest/api/sbt/OutputStrategy.html

# Forking

By default, the `run` action runs in the same JVM as sbt.  Forking is required under [[certain circumstances|Running Project Code]], however.  Or, you might want to fork Java processes when implementing new tasks.

By default, a forked process uses the same Java and Scala versions being used for the build and the working directory and JVM options of the current process.  This page discusses how to enable and configure forking.

# Enable forking

The following examples demonstrate forking the `run` action and changing the working directory or arguments.

To enable forking all `run`-like methods (`run`, `run-main`, `test:run`, and `test:run-main`), set `fork` to `true`.
```scala
fork in run := true
```

To only fork `test:run` and `test:run-main`:
```scala
fork in (Test,run) := true
```

Similarly, set `fork in Compile := true` to only fork the main `run` tasks.  `run` and `run-main` share the same configuration and cannot be configured separately.

# Change working directory

To change the working directory when forked, set `baseDirectory in run` or `baseDirectory in (Test, run)`:
```scala
// sets the working directory for all `run`-like tasks
baseDirectory in run := file("/path/to/working/directory/")

// sets the working directory for `run` and `run-main` only
baseDirectory in (Compile,run) := file("/path/to/working/directory/")

// sets the working directory for `test:run` and `test:run-main` only
baseDirectory in (Test,run) := file("/path/to/working/directory/")
```

# Forked JVM options

To specify options to be provided to the forked JVM, set `javaOptions`:
```scala
javaOptions in run += "-Xmx8G"
```

or specify the configuration to affect only the main or test `run` tasks:
```scala
javaOptions in (Test,run) += "-Xmx8G"
```

# Java Home

Select the Java installation to use by setting the `java-home` directory:
```scala
javaHome := file("/path/to/jre/")
```

Note that if this is set globally, it also sets the Java installation used to compile Java sources.  You can restrict it to running only by setting it in the `run` scope:
```scala
javaHome in run := file("/path/to/jre/")
```

As with the other settings, you can specify the configuration to affect only the main or test `run` tasks.

# Configuring output

By default, forked output is sent to the Logger, with standard output logged at the `Info` level and standard error at the `Error` level.
This can be configured with the `output-strategy` setting, which is of type [OutputStrategy].

```scala
// send output to the build's standard output and error
outputStrategy := Some(StdoutOutput)

// send output to the provided OutputStream `someStream`
outputStrategy := Some(CustomOutput(someStream: OutputStream))

// send output to the provided Logger `log` (unbuffered)
outputStrategy := Some(LoggedOutput(log: Logger))

// send output to the provided Logger `log` after the process terminates
outputStrategy := Some(BufferedOutput(log: Logger))
```

As with other settings, this can be configured individually for main or test `run` tasks.

## Direct Usage

To fork a new Java process, use the [Fork API].  The methods of interest are `Fork.java`, `Fork.javac`, `Fork.scala`, and `Fork.scalac`.  See the [ForkJava] and [ForkScala] classes for the arguments and types.
