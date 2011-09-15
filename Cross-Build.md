# Cross-building

# Introduction

Different versions of Scala can be binary incompatible, despite maintaining source compatibility.  This page describes how to use `sbt` to build and publish your project against multiple versions of Scala and how to use libraries that have done the same.

# Publishing Conventions

The underlying mechanism used to indicate which version of Scala a library was compiled against is to append `_<scala-version>` to the library's name.  For example, `dispatch` becomes `dispatch_2.8.1` for the variant compiled against Scala 2.8.1.  This fairly simple approach allows interoperability with users of Maven, Ant and other build tools.

The rest of this page describes how `sbt` handles this for you as part of cross-building.

# Using Cross-Built Libraries

To use a library built against multiple versions of Scala, double the first `%` in an inline dependency to be `%%`.  This tells `sbt` that it should append the current version of Scala being used to build the library to the dependency's name.  For example:

```scala
  libraryDependencies += "net.databinder" %% "dispatch" % "0.8.0"
```

A nearly equivalent, manual alternative for a fixed version of Scala is:

```scala
  libraryDependencies += "net.databinder" % "dispatch_2.8.1" % "0.8.0"
```

# Cross-Building a Project

Define the versions of Scala to build against in the `cross-scala-versions` setting.  Versions of Scala 2.8.0 or later are allowed.  For example, in a `.sbt` build definition:

```scala
crossScalaVersions := Seq("2.8.0", "2.8.1", "2.9.1")
```

To build against all versions listed in `build.scala.versions`, prefix the action to run with `+`.  For example:

```text
> + package
```

A typical way to use this feature is to do development on a single Scala version (no `+` prefix) and then cross-build (using `+`) occasionally and when releasing.  The ultimate purpose of `+` is to cross-publish your project.  That is, by doing:

```text
> + publish
```

you make your project available to users for different versions of Scala.  See [[Publishing]] for more details on publishing your project.

In order to make this process as quick as possible, different output and managed dependency directories are used for different versions of Scala.  For example, when building against Scala 2.8.1,

* `./target/` becomes `./target/scala_2.8.1/`
* `./lib_managed/` becomes `./lib_managed/scala_2.8.1/`

Packaged jars, wars, and other artifacts have `_<scala-version>` appended to the normal artifact ID as mentioned in the Publishing Conventions section above.

This means that the outputs of each build against each version of Scala are independent of the others.  `sbt` will resolve your dependencies for each version separately.  This way, for example, you get the version of Dispatch compiled against 2.8.1 for your 2.8.1 build, the version compiled against 2.8.0 for your 2.8.0 build, and so on.  In fact, you can control your dependencies for different Scala versions.  For example:

```scala
libraryDependencies <<= (scalaVersion, libraryDependencies) { (sv, deps) =>
		// select the ScalaCheck version based on the Scala version
	val versionMap = Map("2.8.0" -> "1.7", "2.8.1" => "1.8")
	val testVersion = versionMap.getOrElse(sv, error("Unsupported Scala version " + sv))
		// append the ScalaCheck dependency to the existing dependencies
	deps :+ ("org.scala-tools.testing" % "scalacheck" % testVersion)
}
```

This works because your project definition is reloaded for each version of Scala you are building against.  `scalaVersion` contains the current version of Scala being used to build the project.

As a final note, you can use `++ <version>` to temporarily switch the Scala version currently being used to build (see [[Running]] for details).
