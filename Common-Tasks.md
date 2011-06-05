[mailing list]: http://groups.google.com/group/simple-build-tool

# Common Tasks

This page discusses how to handle common tasks.  If you don't see your use case here, please suggest it for inclusion on the [mailing list] or ask how to do it and add it yourself.

# Source and resource generation

sbt provides standard hooks for adding source or resource generation tasks.  A generation task should generate sources in a subdirectory of `sourceManaged` for sources or `resourceManaged` for resources and return a sequence of files generated.  The key to add the task to is called `sourceGenerators` for sources and `resourceGenerators` for resources.  It should be scoped according to whether the generated files are main (`Compile`) or test (`Test`) sources or resources.  This basic structure looks like:

```scala
sourceGenerators in Compile <+= <your Task[Seq[File]] here>
```

For example, assuming a method `def makeSomeSources(base: File): Seq[File]`,

```scala
sourceGenerators in Compile <+= sourceManaged in Compile { outDir: File =>
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

The files included in an artifact are configured by default by a task `mappings` that is scoped by the relevant package task.  This task provides a sequence `Seq[(File,String)]` of mappings from the file to include to the path within the jar.  See [Mapping Files] for details on creating these mappings.

For example, to add generated sources to the packaged source artifact:
```scala
mappings in packageSrc <++= (managedSource, managedSources) map { (base, srcs) =>
    import Path.{flat, relativeTo}
  srcs x (relativeTo(base) | flat)
}
```

This takes sources from the `managedSources` task and relativizes them against the `managedSource` base directory, falling back to a flattened mapping.  If a source generation task doesn't write the sources to the `managedSource` directory, the mapping function would have to be adjusted to try relativizing against additional directories or something more appropriate for the generator.