[mailing list]: http://groups.google.com/group/simple-build-tool/topics
[issue tracker]: https://github.com/harrah/xsbt/issues
[migration page]: https://github.com/harrah/xsbt/wiki/Migrating-from-SBT-0.7.x-to-0.10.x
[checksum report]: https://issues.sonatype.org/browse/MVNCENTRAL-46
[original proposal]: https://gist.github.com/404272
[API Documentation]: http://harrah.github.com/xsbt/latest/api/index.html
[hyperlinked sources]: http://harrah.github.com/xsbt/latest/sxr/index.html

# Frequently Asked Questions

## Project Information

### How do I get help?

Please use the [mailing list] for questions, comments, and discussions.

* Please state the problem or question clearly and provide enough context.  Code examples and build transcripts are often useful when appropriately edited.
* Providing small, reproducible examples are a good way to get help quickly.
* Include relevant information such as the version of sbt and Scala being used.

### How do I report a bug?

Please use the [issue tracker] to report confirmed bugs.  Do not use it to ask questions.  If you are uncertain whether something is a bug, please ask on the [mailing list] first.

### How do I migrate from 0.7 to 0.10?

See the [migration page].

### How can I help?

* Fix mistakes that you notice on the wiki.
* Make [bug reports][issue tracker] that are clear and reproducible.
* Answer questions on the [mailing list].
* Fix issues that affect you.  [Fork, fix, and submit a pull request](http://help.github.com/fork-a-repo/).
* Implement features that are important to you.  There is an [[Opportunities]] page for some ideas, but the most useful contributions are usually ones you want yourself.

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

The invalid checksum should generally be reported to the repository owner (as [was done][checksum report] for the above error).  In the meantime, you can temporarily disable checking with the following setting:

```scala
checksums := Nil
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
