# Plugins Best Practices

_This page is intended primarily for SBT plugin authors._

A plugin developer should strive for consistency and ease of use. Specifically:

* Plugins should play well with other plugins. Avoiding namespace clashes (in both SBT and Scala) is paramount.
* Plugins should follow consistent conventions. The experiences of an SBT _user_ should be consistent, no matter
  what plugins are pulled in.

Here are some current plugin best practices. **NOTE:** Best practices are evolving, so check back frequently.

## Reuse existing keys

SBT has a number of [predefined keys](http://harrah.github.com/xsbt/latest/api/sbt/Keys%24.html). Where possible, reuse them in your plugin. For instance, don't define:

```scala
val sourceFiles = SettingKey[Seq[File]]("source-files")
```

Instead, simply reuse SBT's existing `sources` key.

## Avoid namespace clashes

Sometimes, you need a new key, because there is no existing SBT key. In this case, use a plugin-specific prefix, both in the (string) key name used in the SBT namespace and in the Scala `val`. There are two acceptable ways to accomplish this goal:

### Just use a `val` prefix

```scala
object FooPlugin extends sbt.Plugin {
  val FooConfig = config("foo")
  val fooOutputDirectory = SettingKey[File]("foo-output-dir")
  val fooStylesheet = SettingKey[File]("foo-stylesheet")
}
```

In this approach, every `val` starts with `foo`. A user of the plugin would refer to the settings like this:

```
sources in FooConfig <<= ...
fooOutputDirectory in FooConfig <<= ...
```

### Use a nested object

```scala
object FooPlugin extends sbt.Plugin {
  object foo {
    val Config = config("foo")
    val outputDirectory = SettingKey[File]("foo-output-dir")
    val stylesheet = SettingKey[File]("foo-stylesheet")
  }
}
```

In this approach, all non-common settings are in a nested object. A user of the plugin would refer to the settings like this:

```
foo.outputDirectory in foo.Config <<= ...
sources in foo.Config <<= ...
```

