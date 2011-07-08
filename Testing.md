# Testing

# Basics

The standard source locations for testing are:

* Scala sources in `src/test/scala/`
* Java sources in `src/test/java/`
* Resources for the test classpath in `src/test/resources/`

The resources may be accessed from tests by using the `getResource` methods of `java.lang.Class` or `java.lang.ClassLoader`.

The main Scala testing frameworks (Specs(2), ScalaCheck, and ScalaTest) provide an implementation of the common test interface and only need to be added to the classpath to work with sbt.  For example, ScalaCheck may be used by declaring it as a [[managed dependency|Library Management]]:

```scala
libraryDependencies += "org.scala-tools.testing" %% "scalacheck" % "1.9" % "test"
```

The fourth component `"test"` is the [[configuration|Configurations]] and means that ScalaCheck will only be on the test classpath and it isn't needed by the main sources.
This is generally good practice for libraries because your users don't typically need your test dependencies to use your library.

With the library dependency defined, you can then add test sources in the locations listed above and compile and run tests.
The tasks for running tests are `test` and `test-only`.
The `test` task accepts no command line arguments and runs all tests:

```text
> test
```

## test-only

The `test-only` task accepts a whitespace separated list of test names to run.  For example:
```text
> test-only org.example.MyTest1 org.example.MyTest2
```

It supports wildcards as well:
```text
> test-only org.example.*Slow org.example.MyTest1
```

### Tab completion

Tab completion is provided for test names based on the results of the last `test:compile`.  This means that a new sources aren't available for tab completion until they are compiled and deleted sources won't be removed from tab completion until a recompile.  A new test source can still be manually written out and run using `test-only`.

## Other tasks

Tasks that are available for main sources are generally available for test sources, but are prefixed with `test:` on the command line and are referenced in Scala code with `in Test`.  These tasks include:

* `test:compile`
* `test:console`
* `test:console-quick`
* `test:run`
* `test:run-main`

See [[Running]] for details on these tasks.

# Output

By default, logging is buffered for each test source file until all tests for that file complete.
This can be disabled with:

```scala
logBuffered in Test := false
```

# Options

## Test Framework Arguments

Arguments to the test framework may be provided on the command line to the `test-only` tasks following a `--` separator.  For example:

```text
> test-only org.example.MyTest -- -d -S

```

To specify test framework arguments as part of the build, add options constructed by `Tests.Argument`:

```scala
testOptions in Test += Tests.Argument("-d", "-g")
```

To specify them for a specific test framework only:

```scala
testOptions in Test += Tests.Argument(TestFrameworks.ScalaCheck, "-d", "-g")
```

## Setup and Cleanup

Specify setup and cleanup actions using `Tests.Setup` and `Tests.Cleanup`.
These accept either a function of type `() => Unit` or a function of type `ClassLoader => Unit`.
The variant that accepts a ClassLoader is passed the class loader that is (or was) used for running the tests.
It provides access to the test classes as well as the test framework classes.

Examples:
```scala
testOptions in Test += Tests.Setup( () => println("Setup") )

testOptions in Test += Tests.Cleanup( () => println("Cleanup") )

testOptions in Test += Tests.Setup( loader => ... )

testOptions in Test += Tests.Cleanup( loader => ... )
```

## Disable Parallel Execution of Tests

By default, sbt runs all tasks in parallel.  Because each test is mapped to a task, tests are also run in parallel by default.  To disable parallel execution of tests:

```scala
parallelExecution in Test := false
```
`Test` can be replaced with `IntegrationTest` to only execute integration tests serially.

## Filter classes

If you want to only run test classes whose name ends with "Test", use `Tests.Filter`:

```
testOptions in Test := Seq(Tests.Filter(s => s.endsWith("Test")))
```

# Additional test configurations

You can add an additional test configuration to have a separate set of test sources and associated compilation, packaging, and testing tasks and settings.
The steps are:

* Define the configuration
* Add the tasks and settings
* Declare library dependencies
* Create sources
* Run tasks

The following two examples demonstrate this.
The first example shows how to enable integration tests.
The second shows how to define a customized test configuration.
This allows you to define multiple types of tests per project.

## Integration Tests

The following full build configuration demonstrates integration tests.

```scala
  import sbt._
  import Keys._

object B extends Build
{
  lazy val root =
    Project("root", file("."))
      .configs( IntegrationTest )
      .settings( Defaults.itSettings : _*)
      .settings( libraryDependencies += specs )

  lazy val specs = "org.scala-tools.testing" %% "specs" % "1.6.8" % "it,test"
}
```

* `configs(IntegrationTest)` adds the predefined integration test configuration.  This configuration is referred to by the name `it`.
* `settings( Defaults.itSettings : _* )` adds compilation, packaging, and testing actions and settings in the `IntegrationTest` configuration.
* `settings( libraryDependencies += specs )` adds specs to both the standard `test` configuration and the integration test configuration `it`.  To define a dependency only for integration tests, use `"it"` as the configuration instead of `"it,test"`.

The standard source hierarchy is used:

 * `src/it/scala` for Scala sources
 * `src/it/java` for Java sources
 * `src/it/resources` for resources that should go on the integration test classpath

The standard testing tasks are available, but must be prefixed with `it:`.  For example,

```text
> it:test-only org.example.AnIntegrationTest
```

Similarly the standard settings may be configured for the `IntegrationTest` configuration.
If not specified directly, most `IntegrationTest` settings delegate to `Test` settings by default.
For example, if test options are specified as:

```scala
testOptions in Test += ...
```

then these will be picked up by the `Test` configuration and in turn by the `IntegrationTest` configuration.
Options can be added specifically for integration tests by putting them in the `IntegrationTest` configuration:

```scala
testOptions in IntegrationTest += ...
```

Or, use `:=` to overwrite any existing options, declaring these to be the definitive integration test options:

```scala
testOptions in IntegrationTest := Seq(...)
```

## Custom test configuration

The previous example may be generalized to a custom test configuration.

```scala
  import sbt._
  import Keys._

object B extends Build
{
  lazy val root =
    Project("root", file("."))
      .configs( FunTest )
      .settings( inConfig(FunTest)(Defaults.testSettings) : _*)
      .settings( libraryDependencies += specs )

  lazy val FunTest = config("fun") extend(Test)
  lazy val specs = "org.scala-tools.testing" %% "specs" % "1.6.8" % "fun"
}
```

Instead of using the builtin configuration, we defined a new one:

```scala
lazy val FunTest = config("fun") extend(Test)
```

The `extend(Test)` part means to delegate to `Test` for undefined `CustomTest` settings.
The line that adds the tasks and settings for the new test configuration is:

```scala
settings( inConfig(FunTest)(Defaults.testSettings) : _*)
```

This says to add test and settings tasks in the `FunTest` configuration.
We could have done it this way for integration tests as well.
In fact, `Defaults.itSettings` is a convenience definition: `val itSettings = inConfig(IntegrationTest)(Defaults.testSettings)`.

The comments in the integration test section hold, except with `IntegrationTest` replaced with `FunTest` and `"it"` replaced with `"fun"`.  For example, test options can be configured specifically for `FunTest`:

```scala
testOptions in FunTest += ...
```

Test tasks are run by prefixing them with `fun:`

```scala
> fun:test
```

## Additional test configurations with shared sources

An alternative to adding separate sets of test sources (and compilations) is to share sources.
In this approach, the sources are compiled together using the same classpath and are packaged together.
However, different tests are run depending on the configuration.

```scala
  import sbt._
  import Keys._

object B extends Build
{
  lazy val root =
    Project("root", file("."))
      .configs( FunTest )
      .settings( inConfig(FunTest)(Defaults.testTasks) : _*)
      .settings(
         libraryDependencies += specs,
         testOptions in Test := TestFilter(itFilter),
         testOptions in FunTest := TestFilter(unitFilter)
		 )

  def itFilter(name: String): Boolean = name endsWith "ITest"
  def unitFilter(name: String): Boolean = (name endsWith "Test") && !itFilter(name)

  lazy val FunTest = config("fun") extend(Test)
  lazy val specs = "org.scala-tools.testing" %% "specs" % "1.6.8" % "test"
}
```

The key differences are:

* We are now only adding the test tasks (`inConfig(FunTest)(Defaults.testTasks)`) and not compilation and packaging tasks and settings.
* We filter the tests to be run for each configuration.

To run standard unit tests, run `test` (or equivalently, `test:test`):

```text
> test
```

To run tests for the added configuration (here, `"fun"`), prefix it with the configuration name as before:

```text
> fun:test
> fun:test-only org.example.AFunTest
```

### Application to parallel execution

One use for this shared-source approach is to separate tests that can run in parallel from those that must execute serially.
Appy the procedure described in this section for an additional configuration.
Let's call the configuration `serial`:

```scala
  lazy val Serial = config("serial") extend(Test)
```

Then, we can disable parallel execution in just that configuration using:
```text
parallelExecution in Serial := false
```

The tests to run in parallel would be run with `test` and the ones to run in serial would be run with `serial:test`.


# JUnit

Support for JUnit is provided by [junit-interface](https://github.com/szeiger/junit-interface).  To add JUnit support into your project, add the junit-interface dependency in your project's main build.sbt file.
```test
libraryDependencies += "com.novocode" % "junit-interface" % "0.7" % "test->default"
```
