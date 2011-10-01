[Apache Ivy]: http://ant.apache.org/ivy/
[Ivy revisions]: http://ant.apache.org/ivy/history/2.2.0/ivyfile/dependency.html#revision
[Extra attributes]: http://ant.apache.org/ivy/history/2.2.0/concept.html#extra
[through Ivy]: http://ant.apache.org/ivy/history/latest-milestone/concept.html#checksum

# Library Management

# Introduction

There are two ways for you to manage libraries with sbt: manually or automatically.  These two ways can be mixed as well.  This page discusses the two approaches.  All configurations shown here are settings that go either directly in a [[Basic Configuration]] or are appended to the `settings` of a Project in a [[Full Configuration]].

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

# Automatic Dependency Management

This method of dependency management involves specifying the direct dependencies of your project and letting sbt handle retrieving and updating your dependencies.  sbt supports three ways of specifying these dependencies:

* Declarations in your project definition
* Maven POM files (dependency definitions only: no repositories)
* Ivy configuration and settings files

sbt uses [Apache Ivy] to implement dependency management in all three cases.  The default is to use inline declarations, but external configuration can be explicitly selected.  The following sections describe how to use each method of automatic dependency management.

## Inline Declarations

Inline declarations are a basic way of specifying the dependencies to be automatically retrieved.  They are intended as a lightweight alternative to a full configuration using Ivy.

### Dependencies

Declaring a dependency looks like:

```scala
libraryDependencies += groupID % artifactID % revision
```

or

```scala
libraryDependencies += groupID % artifactID % revision % configuration
```

See [[Configurations]] for details on configuration mappings.  Also, several dependencies can be declared together:

```scala
libraryDependencies ++= Seq(
	groupID %% artifactID % revision,
	groupID %% otherID % otherRevision
)
```

If you are using a dependency that was built with sbt, double the first `%` to be `%%`:

```scala
libraryDependencies += groupID %% artifactID % revision
```

This will use the right jar for the dependency built with the version of Scala that you are currently using.  If you get an error while resolving this kind of dependency, that dependency probably wasn't published for the version of Scala you are using.  See [[Cross Build]] for details.

Ivy can select the latest revision of a module according to constraints you specify.  Instead of a fixed revision like `"1.6.1"`, you specify `"latest.integration"`, `"2.9.+"`, or `"[1.0,)"`.  See the [Ivy revisions] documentation for details.


### Resolvers
 
sbt uses the standard Maven2 repository and the Scala Tools Releases (<http://scala-tools.org/repo-releases>) repositories by default.

Declare additional repositories with the form:

```scala
resolvers += name at location
```

For example:

```scala
libraryDependencies ++= Seq(
	"org.apache.derby" % "derby" % "10.4.1.3",
	"org.specs" % "specs" % "1.6.1"
)

resolvers += "Scala-Tools Maven2 Snapshots Repository" at "http://scala-tools.org/repo-snapshots"
```

The scala-tools.org snapshots repository can be referenced a bit more conveniently:

```scala
resolvers += ScalaToolsSnapshots
```

sbt can search your local Maven repository if you add it as a repository:

```scala
resolvers += "Local Maven Repository" at "file://"+Path.userHome.absolutePath+"/.m2/repository"
```

See [[Resolvers]] for details on defining other types of repositories.

### Override default resolvers

`resolvers` configures additional, inline user resolvers.  By default, `sbt` combines these resolvers with default repositories (Maven Central, Scala Tools, and the local Ivy repository) to form `external-resolvers`.  To have more control over repositories, set `external-resolvers` directly.  To only specify repositories in addition to the usual defaults, configure `resolvers`.

For example, to use the Scala Tools snapshots repository in addition to the default repositories,

```scala
resolvers += ScalaToolsSnapshots
```

To use the local and Maven Central repositories, but not the Scala Tools releases repository:

```scala
externalResolvers <<= resolvers map { rs =>
  Resolver.withDefaultResolvers(rs, mavenCentral = true, scalaTools = false)
}
```

For complete control, configure `full-resolvers`.  This should rarely be modified, however, because `full-resolvers` combines `project-resolver` with `external-resolvers`.  `project-resolver` is used for inter-project dependency management and should (almost) always be included.

### Explicit URL

If your project requires a dependency that is not present in a repository, a direct URL to its jar can be specified as follows:

```scala
libraryDependencies += "slinky" % "slinky" % "2.1" from "http://slinky2.googlecode.com/svn/artifacts/2.1/slinky.jar"
```

The URL is only used as a fallback if the dependency cannot be found through the configured repositories.  Also, the explicit URL is not included in published metadata (that is, the pom or ivy.xml).

### Disable Transitivity

By default, these declarations fetch all project dependencies, transitively. In some instances, you may find that the dependencies listed for a project aren't necessary for it to build.  Projects using the Felix OSGI framework, for instance, only explicitly require its main jar to compile and run. Avoid fetching artifact dependencies with either `intransitive()` or `notTransitive()`, as in this example:

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
This can be configured machine-wide, for use by both the sbt launcher and by projects, by setting the system property `sbt.ivy.home` in the sbt startup script (described in [[Setup]]).

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

* For Ivy settings (resolver configuration), use `externalIvySettings()` or
 `externalIvySettings(baseDirectory(_ / "custom-settings-name.xml"))`.
* For an Ivy file (dependency configuration), use `externalIvyFile()` or
 `externalIvyFile(baseDirectory(_ / "custom-name.xml"))`.
* For a Maven pom (dependencies only), use `externalPom()` or
 `externalPom(baseDirectory(_ / "custom-name.xml"))`.

For example, a `build.sbt` using external Ivy files might look like:

```scala
externalIvyFile( baseDirectory { base => base / "ivyA.xml"} )

externalIvySettings()
```

Maven support is dependent on Ivy's support for Maven POMs.
Known issues with this support:

* Specifying `relativePath` in the `parent` section of a POM will produce an error.
* Ivy ignores repositories specified in the POM.  A workaround is to specify repositories inline or in an Ivy `ivysettings.xml` file.