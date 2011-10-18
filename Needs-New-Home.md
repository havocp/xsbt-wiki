# Snippets of docs that need to move to another page



Temporarily change the logging level and configure how stack traces are displayed by modifying the `log-level` or `trace-level` settings:

```text
> set logLevel := Level.Warn
```

Valid `Level` values are `Debug, Info, Warn, Error`.

You can run an action for multiple versions of Scala by prefixing the action with `+`.  See [[Cross Build]] for details.  You can temporarily switch to another version of Scala using `++ <version>`.  This version does not have to be listed in your build definition, but it does have to be available in a repository.  You can also include the initial command to run after switching to that version.  For example:

```text
> ++2.9.1 console-quick
...
Welcome to Scala version 2.9.1.final (Java HotSpot(TM) Server VM, Java 1.6.0).
...
scala>
...
> ++2.8.1 console-quick
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
  `reload plugins` changes the current project to the build definition project (in `project/`).  This can be useful to directly manipulate the build definition.  For example, running `clean` on the build definition project will force snapshots to be updated and the build definition to be recompiled.
  `reload return` changes back to the main project.
* `set <setting-expression>`
  Evaluates and applies the given setting definition.  The setting applies until sbt is restarted, the build is reloaded, or the setting is overridden by another `set` command or removed by the `session` command.  See [[Basic Configuration]] and [[Inspecting Settings]] for details.
* `session <command>`
  Manages session settings defined by the `set` command.  See [[Inspecting Settings]] for details.
* `inspect <setting-key>`
  Displays information about settings, such as the value, description, defining scope, dependencies, delegation chain, and related settings.  See [[Inspecting Settings]] for details.


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



# Manual Dependency Management

Manually managing dependencies involves copying any jars that you want to use to the `lib` directory.  sbt will put these jars on the classpath during compilation, testing, running, and when using the interpreter.  You are responsible for adding, removing, updating, and otherwise managing the jars in this directory.  No modifications to your project definition are required to use this method unless you would like to change the location of the directory you store the jars in.

To change the directory jars are stored in, change the `unmanaged-base` setting in your project definition.  For example, to use `custom_lib/`:

```scala
unmanagedBase <<= baseDirectory { base => base / "custom_lib" }
```

If you want more control and flexibility, override the `unmanaged-jars` task, which ultimately provides the manual dependencies to sbt.  The default implementation is roughly:

```scala
unmanagedJars in Compile <<= baseDirectory map { base => (base ** "*.jar").classpath }
```

If you want to add jars from multiple directories in addition to the default directory, you can do:

```scala
unmanagedJars in Compile <++= baseDirectory map { base =>
	val baseDirectories = (base / "libA") +++ (base / "b" / "lib") +++ (base / "libC")
	val customJars = (baseDirectories ** "*.jar") +++ (base / "d" / "my.jar")
	customJars.classpath
}
```

See [[Paths]] for more information on building up paths.



### Resolver.withDefaultResolvers method

To use the local and Maven Central repositories, but not the Scala Tools releases repository:

```scala
externalResolvers <<= resolvers map { rs =>
  Resolver.withDefaultResolvers(rs, mavenCentral = true, scalaTools = false)
}
```


### Explicit URL

If your project requires a dependency that is not present in a repository, a
direct URL to its jar can be specified with the `from` method as follows:

```scala
libraryDependencies += "slinky" % "slinky" % "2.1" from "http://slinky2.googlecode.com/svn/artifacts/2.1/slinky.jar"
```

The URL is only used as a fallback if the dependency cannot be found through
the configured repositories. Also, when you publish a project, a pom or
ivy.xml is created listing your dependencies; the explicit URL is not
included in this published metadata.

### Disable Transitivity

By default, sbt fetches all dependencies, transitively. (That is, it downloads
the dependencies of the dependencies you list.)

In some instances, you may find that the dependencies listed for a project
aren't necessary for it to build. Avoid fetching artifact dependencies with
`intransitive()`, as in this example:

```scala
libraryDependencies += "org.apache.felix" % "org.apache.felix.framework" % "1.8.0" intransitive()
```

### Classifiers

You can specify the classifer for a dependency using the `classifier` method.  For example, to get the jdk15 version of TestNG:

```scala
libraryDependencies += "org.testng" % "testng" % "5.7" classifier "jdk15"
```

To obtain particular classifiers for all dependencies transitively, run the `update-classifiers` task.  By default, this resolves all artifacts with the `sources` or `javadoc` classifer.  Select the classifiers to obtain by configuring the `transitive-classifiers` setting.  For example, to only retrieve sources:

```scala
transitiveClassifiers := Seq("sources")
```

### Extra Attributes

[Extra attributes] can be specified by passing key/value pairs to the `extra` method.

To select dependencies by extra attributes:

```scala
libraryDependencies += "org" % "name" % "rev" extra("color" -> "blue")
```

To define extra attributes on the current project:

```scala
projectID <<= projectID { id =>
    id extra("color" -> "blue", "component" -> "compiler-interface")
}
```

### Inline Ivy XML

sbt additionally supports directly specifying the configurations or dependencies sections of an Ivy configuration file inline.  You can mix this with inline Scala dependency and repository declarations.

For example:

```scala
ivyXML :=
  <dependencies>
    <dependency org="javax.mail" name="mail" rev="1.4.2">
      <exclude module="activation"/>
    </dependency>
  </dependencies>
```

### Ivy Home Directory

By default, sbt uses the standard Ivy home directory location `${user.home}/.ivy2/`.
This can be configured machine-wide, for use by both the sbt launcher and by projects, by setting the system property `sbt.ivy.home` in the sbt startup script (described in [[Setup|Getting Started Setup]]).

For example:

```text
java -Dsbt.ivy.home=/tmp/.ivy2/ ...
```

### Checksums

sbt ([through Ivy]) verifies the checksums of downloaded files by default.  It also publishes checksums of artifacts by default.  The checksums to use are specified by the _checksums_ setting.

To disable checksum checking during update:

```scala
checksums in update := Nil
```

To disable checksum creation during artifact publishing:

```scala
checksums in publishLocal := Nil

checksums in publish := Nil
```

The default value is:

```scala
checksums := Seq("sha1", "md5")
```

### Publishing

Finally, see [[Publishing]] for how to publish your project.

## Maven/Ivy

For this method, create the configuration files as you would for Maven (`pom.xml`) or Ivy (`ivy.xml` and optionally `ivysettings.xml`).
External configuration is selected by using one of the following expressions.

### Ivy settings (resolver configuration)

```scala
externalIvySettings()
```

or

```scala
externalIvySettings(baseDirectory(_ / "custom-settings-name.xml"))
```

### Ivy file (dependency configuration)

```scala
externalIvyFile()
```

or

```scala
externalIvyFile(baseDirectory(_ / "custom-name.xml"))
```

Because Ivy files specify their own configurations, sbt needs to know which configurations to use for the compile, runtime, and test classpaths.  For example, to specify that the Compile classpath should use the 'default' configuration:

```scala
classpathConfiguration in Compile := config("default")
```

### Maven pom (dependencies only)

```scala
externalPom()
```

or

```scala
externalPom(baseDirectory(_ / "custom-name.xml"))
```

### Full Ivy Example

For example, a `build.sbt` using external Ivy files might look like:

```scala
externalIvySettings()

externalIvyFile( baseDirectory { base => base / "ivyA.xml"} )

classpathConfiguration in Compile := Compile

classpathConfiguration in Test := Test

classpathConfiguration in Runtime := Runtime
```

### Known limitations

Maven support is dependent on Ivy's support for Maven POMs.
Known issues with this support:

* Specifying `relativePath` in the `parent` section of a POM will produce an error.
* Ivy ignores repositories specified in the POM.  A workaround is to specify repositories inline or in an Ivy `ivysettings.xml` file.
