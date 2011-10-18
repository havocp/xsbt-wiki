[mailing list]: http://groups.google.com/group/simple-build-tool/topics
[issue tracker]: https://github.com/harrah/xsbt/issues
[migration page]: https://github.com/harrah/xsbt/wiki/Migrating-from-SBT-0.7.x-to-0.10.x
[checksum report]: https://issues.sonatype.org/browse/MVNCENTRAL-46
[original proposal]: https://gist.github.com/404272
[API Documentation]: http://harrah.github.com/xsbt/latest/api/index.html
[hyperlinked sources]: http://harrah.github.com/xsbt/latest/sxr/index.html
[sbt-launch.jar]: http://repo.typesafe.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.11.0/sbt-launch.jar
[mailing list]: http://groups.google.com/group/simple-build-tool/
[xsbt-web-plugin]: https://github.com/siasia/xsbt-web-plugin
[xsbt-webstart]: https://github.com/ritschwumm/xsbt-webstart
[FileUtilities]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/FileUtilities$object.html
[IO]: http://harrah.github.com/xsbt/latest/api/sbt/IO$.html
[Path 0.7]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/Path.html
[Path object]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/Path$.html
[Path 0.11]: http://harrah.github.com/xsbt/latest/api/sbt/Path$.html
[PathFinder 0.7]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/PathFinder.html
[PathFinder object]: http://harrah.github.com/xsbt/latest/api/sbt/PathFinder$.html
[PathFinder 0.11]: http://harrah.github.com/xsbt/latest/api/sbt/PathFinder.html
[RichFile]: http://harrah.github.com/xsbt/latest/api/sbt/RichFile.html

# Frequently Asked Questions

## Project Information

### How do I get help?

Please use the [mailing list] for questions, comments, and discussions.

* Please state the problem or question clearly and provide enough context.  Code examples and build transcripts are often useful when appropriately edited.
* Providing small, reproducible examples are a good way to get help quickly.
* Include relevant information such as the version of sbt and Scala being used.

### How do I report a bug?

Please use the [issue tracker] to report confirmed bugs.  Do not use it to ask questions.  If you are uncertain whether something is a bug, please ask on the [mailing list] first.

### How can I help?

* Fix mistakes that you notice on the wiki.
* Make [bug reports][issue tracker] that are clear and reproducible.
* Answer questions on the [mailing list].
* Fix issues that affect you.  [Fork, fix, and submit a pull request](http://help.github.com/fork-a-repo/).
* Implement features that are important to you.  There is an [[Opportunities]] page for some ideas, but the most useful contributions are usually ones you want yourself.

For more details on developing sbt, see [Developing.pdf](http://harrah.github.com/xsbt/Developing.pdf)

## 0.7 to 0.10+ Migration

### How do I migrate from 0.7 to 0.10+?

See the [[migration page|Migrating-from-SBT-0.7.x-to-0.10.x]]
first and then the following questions.

### Where has 0.7's `lib_managed` gone?

By default, sbt 0.11 loads managed libraries from your ivy cache without copying them to a `lib_managed` directory. This fixes some bugs with the previous solution and keeps your project directory small. If you want to insulate your builds from the ivy cache being cleared, set `retrieveManaged := true` and the dependencies will be copied to `lib_managed` as a build-local cache (while avoiding the issues of `lib_managed` in 0.7.x).

This does mean that existing solutions for sharing libraries with your favoured IDE may not work.  There are 0.11.x plugins for IDEs being developed:

* IntelliJ IDEA: [[https://github.com/mpeltonen/sbt-idea]]
* Netbeans: [[https://github.com/remeniuk/sbt-netbeans-plugin]]
* Eclipse: [[https://github.com/typesafehub/sbteclipse]]

### What are the commands I can use in 0.11 vs. 0.7?

For a list of commands, run `help`.  For details on a specific
command, run `help <command>`.  To view a list of tasks defined on
the current project, run `tasks`.  Alternatively, see the
[[Running|Getting Started Running]] page in the Getting Started Guide for descriptions of common commands and tasks.

If in doubt start by just trying the old command as it may just work.  The built in TAB completion will also assist you, so you can just press TAB at the beginning of a line and see what you get.

The following commands work pretty much as in 0.7 out of the box:

    reload
    update
    compile
    test
    test-only
    publish-local
    exit

### Why have the resolved dependencies in a multi-module project changed since 0.7?

sbt 0.10 fixes a flaw in how dependencies get resolved in multi-module projects.  This change ensures that only one version of a library appears on a classpath.

Use `last update` to view the debugging output for the last `update` run.  Use `show update` to view a summary of files comprising managed classpaths.

### My tests all run really fast but some are broken that weren't in 0.7!

Be aware that compilation and tests run in parallel by default in sbt 0.11. If your test code isn't thread-safe then you may want to change this behaviour by adding one of the following to your `build.sbt`:

```scala
// Execute tests in the current project serially.
// Tests from other projects may still run concurrently.
parallelExecution in Test := false

// Execute everything serially (including compilation and tests)
parallelExecution := false
```

### How do I set log levels in 0.11 vs. 0.7?

`warn`, `info`, `debug` and `error` don't work any more.

The new syntax in the sbt 0.11.x shell is:
```text
> set logLevel := Level.Warn
```

Or in your `build.sbt` file write:

```scala
logLevel := Level.Warn
```

### What happened to the web development and Web Start support since 0.7?

Web application support was split out into a plugin.  See the [xsbt-web-plugin] project.

For an early version of an xsbt Web Start plugin, visit the [xsbt-webstart] project.

### How are inter-project dependencies different in 0.11 vs. 0.7?

In 0.11, there are three types of project dependencies (classpath, execution, and configuration) and they are independently defined.  These were combined in a single dependency type in 0.7.x.  A declaration like:

```scala
lazy val a = project("a", "A")
lazy val b = project("b", "B", a)
```

meant that the `B` project had a classpath and execution dependency on `A` and `A` had a configuration dependency on `B`.  Specifically, in 0.7.x:

1. Classpath: Classpaths for `A` were available on the appropriate classpath for `B`.
1. Execution: A task executed on `B` would be executed on `A` first.
1. Configuration: For some settings, if they were not overridden in `A`, they would default to the value provided in `B`.

In 0.11, declare the specific type of dependency you want. Read
about [[multi-project builds|Getting Started Multi-Project]] in
the Getting Started Guide for details.

### Where did class/object X go since 0.7?

| 0.7 | 0.11 |
| --- | --- |
| [FileUtilities] | [IO] |
| [Path class][Path 0.7] and [object][Path object] | [Path object][Path 0.11], `File`, [RichFile] |
| [PathFinder class][PathFinder 0.7] | `Seq[File]`, [PathFinder class][PathFinder 0.11], [PathFinder object][PathFinder object] |

### Where can I find plugins for 0.11?

See [[sbt 0.10 plugins list]] for a list of currently available plugins.


## Usage

### My last command didn't work but I can't see an explanation. Why?

sbt 0.11 by default suppresses most stack traces and debugging information.  It has the nice side effect of giving you less noise on screen, but as a newcomer it can leave you lost for explanation.  To see the previous output of a command at a higher verbosity, type `last <task>` where `<task>` is the task that failed or that you want to view detailed output for.  For example, if you find that your `update` fails to load all the dependencies as you expect you can enter:

```text
> last update
```

and it will display the full output from the last run of the `update` command.

### How do I disable ansi codes in the output?

Sometimes sbt doesn't detect that ansi codes aren't supported and you get output that looks like:

```
  [0m[ [0minfo [0m]  [0mSet current project to root
```

or ansi codes are supported but you want to disable colored output.  To completely disable ansi codes, set the `sbt.log.noformat` system property to `true`.  For example,

```
sbt -Dsbt.log.noformat=true
```

## Build definitions

### What does `:=`, `%`, ... mean?

See the [[Index]] of commonly used methods, values, and types.  See also the [API Documentation] and the [hyperlinked sources].

### What is `ModuleID`, `Project`, ...?

See the [[Index]] of commonly used methods, values, and types.  See also the [API Documentation] and the [hyperlinked sources].

## Dependency Management

### How do I resolve a checksum error?

This error occurs when the published checksum, such as a sha1 or md5 hash, differs from the checksum computed for a downloaded artifact, such as a jar or pom.xml.  An example of such an error is:

```
[warn]  problem while downloading module descriptor:
http://repo1.maven.org/maven2/commons-fileupload/commons-fileupload/1.2.2/commons-fileupload-1.2.2.pom: 
invalid sha1: expected=ad3fda4adc95eb0d061341228cc94845ddb9a6fe computed=0ce5d4a03b07c8b00ab60252e5cacdc708a4e6d8 (1070ms) 
```

The invalid checksum should generally be reported to the repository owner (as [was done][checksum report] for the above error). In the meantime, you can temporarily disable checking with the following setting:

```scala
checksums in update := Nil
```

See [[Library Management]] for details.

## Miscellaneous

### How do I use the Scala interpreter in my code?

sbt runs tests in the same JVM as sbt itself and Scala classes are not in the same class loader as the application classes.  Therefore, when using the Scala interpreter, it is important to set it up properly to avoid an error message like:

```
 Failed to initialize compiler: class scala.runtime.VolatileBooleanRef not found.
 ** Note that as of 2.8 scala does not assume use of the java classpath.
 ** For the old behavior pass -usejavacp to scala, or if using a Settings
 ** object programatically, settings.usejavacp.value = true.
```

The key is to initialize the Settings for the interpreter using _embeddedDefaults_.  For example:

```scala
 val settings = new Settings
 settings.embeddedDefaults[MyType]
 val interpreter = new Interpreter(settings, ...)
```

Here, MyType is a representative class that should be included on the interpreter's classpath and in its application class loader.  For more background, see the [original proposal] that resulted in _embeddedDefaults_ being added.

Similarly, use a representative class as the type argument when using the _break_ and _breakIf_ methods of _ILoop_, as in the following example:

```scala
  def x(a: Int, b: Int) = {
    import scala.tools.nsc.interpreter.ILoop
    ILoop.breakIf[MyType](a != b, "a" -> a, "b" -> b )
  }
```
