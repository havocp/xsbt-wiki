[Attributed]: http://harrah.github.com/xsbt/latest/api/sbt/Attributed.html

# Classpaths, sources, and resources (Draft)

This page discusses how sbt builds up classpaths for different actions, like `compile`, `run`, and `test` and how to override or augment these classpaths.

# Basics

In sbt 0.10.0 and later, classpaths now include the Scala library and (when declared as a dependency) the Scala compiler.  Classpath-related settings and tasks typically provide a value of type `Classpath`.  This is an alias for `Seq[Attributed[File]]`.  [Attributed] is a type that associates a heterogeneous map with each classpath entry.  Currently, this allows sbt to associate the `Analysis` resulting from compilation with the corresponding classpath entry and for managed entries, the `ModuleID` and `Artifact` that defined the dependency.

To explicitly extract the raw `Seq[File]`, use the `files` method implicitly added to `Classpath`:
```scala
val cp: Classpath = ...
val raw: Seq[File] = cp.files
```

To create a `Classpath` from a `Seq[File]`, use `classpath` and to create an `Attributed[File]` from a `File`, use `Attributed.blank`:

```scala
val raw: Seq[File] = ...
val cp: Classpath = raw.classpath

val rawFile: File = ..
val af: Attributed[File] = Attributed.blank(rawFile)
```

## Unmanaged v. managed

Classpaths, sources, and resources are separated into two main categories: unmanaged and managed.
Unmanaged files are manually created files that are outside of the control of the build.
They are the inputs to the build.
Managed files are under the control of the build.
These include generated sources and resources as well as resolved and retrieved dependencies and compiled classes.

Tasks that produce managed files should be inserted as follows:
```scala
sourceGenerators in Compile <+= sourceManaged in Compile map { out =>
	generate(out / "some_directory")
}
```

In this example, `generate` is some function of type `File => Seq[File]` that actually does the work.
The `<+=` method is like `+=`, but allows the right hand side to have inputs (like the difference between `:=` and `<<=`).
So, we are appending a new task to the list of main source generators (`sourceGenerators in Compile`).

To insert a named task, which is the better approach for plugins:
```scala
sourceGenerators in Compile <+= (mySourceGenerator in Compile).task.identity

mySourceGenerator in Compile <<= sourceManaged in Compile map { out =>
    generate(out / "some_directory")
}
```

where `mySourceGenerator` is defined as:
```scala
val mySourceGenerator = TaskKey[Seq[File]](...)
```

The `task` method is used to refer to the actual task instead of the result of the task.

For resources, there are similar keys `resourceGenerators` and `resourceManaged`.

## External v. internal

Classpaths are also divided into internal and external dependencies.
The internal dependencies are inter-project dependencies.
These effectively put the outputs of one project on the classpath of another project.

External classpaths are the union of the unmanaged and managed classpaths.

## Keys

For classpaths, the relevant keys are:

* `unmanaged-classpath`
* `managed-classpath`
* `external-dependency-classpath`
* `internal-dependency-classpath`

For sources:

* `unmanaged-sources`  These are by default built up from `unmanaged-source-directories`, which consists of `scala-source` and `java-source`.
* `managed-sources`  These are generated sources.  
* `sources` Combines `managed-sources` and `unmanaged-sources`.
* `source-generators` These are tasks that generate source files.  Typically, these tasks will put sources in the directory provided by `source-managed`.

For resources

* `unmanaged-resources`  These are by default built up from `unmanaged-resource-directories`, which by default is `resource-directory`, excluding files matched by `default-excludes`.
* `managed-resources`  By default, this is empty for standard projects.  sbt plugins will have a generated descriptor file here.
* `resource-generators` These are tasks that generate resource files.  Typically, these tasks will put resources in the directory provided by `resource-managed`.

Use the [[inspect command|Inspecting Settings]] for more details.

## Example

You have a standalone project which uses a library that loads xxx.properties from classpath at run time. You put xxx.properties inside directory "config". When you run "sbt run", you want the directory to be in classpath.

```scala

unmanagedClasspath in Runtime <<= (unmanagedClasspath in Runtime, baseDirectory) map { (cp, bd) => cp.:+(new Attributed(bd / "config")(AttributeMap.empty)) }

```