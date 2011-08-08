[mailing list]: http://groups.google.com/group/simple-build-tool

# Common Tasks

This page discusses how to handle common tasks.  If you don't see your use case here, please suggest it for inclusion on the [mailing list] or ask how to do it and add it yourself.

# Additional run tasks

This section is extracted from a [mailing list discussion](http://groups.google.com/group/simple-build-tool/browse_thread/thread/4c28ee5b7e18b46a/).

## Basics

A basic run task is created by: 
```scala
  // this lazy val has to go in a full configuration 
  lazy val myRunTask = TaskKey[Unit]("my-run-task")

  // this can go either in a `build.sbt` or the settings member 
  //   of a Project in a full configuration
  fullRunTask(myRunTask, Test, "foo.Foo", "arg1", "arg2")
```

or, if you really want to define it inline (as in a basic build.sbt file): 

```scala
   fullRunTask(TaskKey[Unit]("my-run-task"), Test, "foo.Foo", "arg1", "arg2") 
```

If you want to be able to supply arguments on the command line, replace `TaskKey` with `InputKey` and `fullRunTask` with `fullRunInputTask`. 
The `Test` part can be replaced with another configuration, such as `Compile`, to use that configuration's classpath.

This run task can be configured individually by specifying the task key in the scope.  For example:

```scala
fork in myRunTask := true

javaOptions in myRunTask += "-Xmx6144m" 
```

## Task Delegation

Something that might be helpful for groups of tasks is defining delegation for tasks. 

The following key definitions specify that settings for `myRun` delegate to `aRun`
```scala
val aRun = TaskKey[Unit]("a-run", "A run task.") 

//   The last parameter to TaskKey.apply here is a repeated one
val myRun = TaskKey[Unit]("my-run", "Custom run task.", aRun)
```

In use, this looks like:
```scala
// Make the run task as before. 
fullRunTask(myRun, Compile, "pkg.Main", "arg1", "arg2") 

// If fork in myRun is not explicitly set, 
//   then this also configures myRun to fork. 
// If fork in myRun is set, it overrides this setting 
//   because it is more specific. 
fork in aRun := true 

// Appends "-Xmx2G" to the current options for myRun. 
//   Because we haven't defined them explicitly, 
//   the current options are delegated to aRun. 
//   So, this says to use the same options as aRun 
//   plus -Xmx2G. 
javaOptions in myRun += "-Xmx2G"
```

# Source and resource generation

sbt provides standard hooks for adding source or resource generation tasks.  A generation task should generate sources in a subdirectory of `sourceManaged` for sources or `resourceManaged` for resources and return a sequence of files generated.  The key to add the task to is called `sourceGenerators` for sources and `resourceGenerators` for resources.  It should be scoped according to whether the generated files are main (`Compile`) or test (`Test`) sources or resources.  This basic structure looks like:

```scala
sourceGenerators in Compile <+= <your Task[Seq[File]] here>
```

For example, assuming a method `def makeSomeSources(base: File): Seq[File]`,

```scala
sourceGenerators in Compile <+= sourceManaged in Compile map { outDir: File =>
  makeSomeSources(outDir / "demo")
}
```

As a specific example, the following generates a hello world source file:

```scala
sourceGenerators in Compile <+= sourceManaged in Compile map { dir =>
  val file = dir / "demo" / "Test.scala"
  IO.write(file, """object Test extends App { println("Hi") }""")
  Seq(file)
}
```

Executing 'run' will print "Hi".  Change `Compile` to `Test` to make it a test source.  To generate resources, change `sourceGenerators` to `resourceGenerators` and `sourceManaged` to `resourceManaged`.  Normally, you would only want to generate sources when necessary and not every run.

By default, generated sources and resources are not included in the packaged source artifact.  To do so, add them as you would other mappings.  See the `Adding files to a package` section.

# Adding files to a package

The files included in an artifact are configured by default by a task `mappings` that is scoped by the relevant package task.  The `mappings` task returns a sequence `Seq[(File,String)]` of mappings from the file to include to the path within the jar.  See [[Mapping Files]] for details on creating these mappings.

For example, to add generated sources to the packaged source artifact:
```scala
mappings in (Compile, packageSrc) <++=
  (sourceManaged in Compile, managedSources in Compile) map { (base, srcs) =>
      import Path.{flat, relativeTo}
    srcs x (relativeTo(base) | flat)
  }
```

This takes sources from the `managedSources` task and relativizes them against the `managedSource` base directory, falling back to a flattened mapping.  If a source generation task doesn't write the sources to the `managedSource` directory, the mapping function would have to be adjusted to try relativizing against additional directories or something more appropriate for the generator.

# Adding a compile configuration

The following example demonstrates adding a new set of compilation settings and tasks to a new configuration called `samples`.  The sources for this configuration go in `src/samples/scala/`.  Unspecified settings delegate to those defined for the `compile` configuration.  For example, if `scalacOptions` are not overridden for `samples`, the options for the main sources are used.

Options specific to `samples` may be declared like:
```scala
scalacOptions in Samples += "-deprecation"
```

This uses the main options as base options because of `+=`.  Use `:=` to ignore the main options:

```scala
scalacOptions in Samples := "-deprecation" :: Nil
```

The example adds all of the usual compilation related settings and tasks to `samples`: 
```text
samples:run
samples:run-main
samples:compile
samples:console
samples:console-quick
samples:scalac-options
samples:full-classpath
samples:package
samples:package-src
...
```

## Example

`project/Sample.scala`

```scala
import sbt._
import Keys._

object Sample extends Build {
     // defines a new configuration "samples" that will delegate to "compile"
   lazy val Samples = config("samples") extend(Compile)

     // defines the project to have the "samples" configuration
   lazy val p = Project("p", file("."))
      .configs(Samples)
      .settings(sampleSettings : _*)

   def sampleSettings = 
        // adds the default compile/run/... tasks in "samples"
      inConfig(Samples)(Defaults.configSettings) ++
      Seq(
        // (optional) makes "test:compile" depend on "samples:compile"
         compile in Test <<= compile in Test dependsOn (compile in Samples)
      ) ++ 
        // (optional) declare that the samples binary and
        // source jars should be published
      publishArtifact(packageBin) ++
      publishArtifact(packageSrc)

   def publishArtifact(task: TaskKey[File]): Seq[Setting[_]] =
      addArtifact(artifact in (Samples, task), task in Samples).settings
}
```

# Adding a new test configuration

See the `Additional test configurations` section of [[Testing]].

# Tool dependencies

Tool dependencies are used to implement a task and are not needed by project source code.  These dependencies can be declared in their own configuration and classpaths.  These are the steps:

1. Define a new [[configuration|Configurations]].
2. Declare the tool [[dependencies|Library Management]] in that configuration.
3. Define a classpath that pulls the dependencies from the [[Update Report]] produced by `update`.
4. Use the classpath to implement the task.

As an example, consider a `proguard` task.  This task needs the ProGuard jars in order to run the tool.  Assuming a new configuration defined in the full build definition (#1):

```scala
val ProguardConfig = config("proguard") hide
```

the following are settings that implement #2-#4:

```scala
// Add proguard as a dependency in the custom configuration.
//  This keeps it separate from project dependencies.
libraryDependencies +=
   "net.sf.proguard" % "proguard" % "4.4" % ProguardConfig.name

// Extract the dependencies from the UpdateReport.
managedClasspath in proguard <<=
   (classpathTypes in proguard, update) map { (ct, report) =>
     Classpaths.managedJars(proguardConfig, ct, report)
   }

// Use the dependencies in a task, typically by putting them
//  in a ClassLoader and reflectively calling an appropriate
//  method.
proguard <<= managedClasspath in proguard { (cp: Seq[File] => 
  // ... do something with 'cp', which includes proguard ...
}
```