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

Sometimes, you need a new key, because there is no existing SBT key. In this case, use a plugin-specific prefix, both in the (string) key name used in the SBT namespace and in the Scala `val`. There are two acceptable ways to accomplish this goal.

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


## Configuration Cat says "Configuration is for configuration" ##

When defining a new type of configuration, e.g.

```scala
val Config = config("profile")
```

should be used to create a "cross-task" configuration.  The task definitions don't change in this case, but the default configuration does.  For example, the `profile` configuration can extend the test configuration with additional settings and changes to allow profiling in SBT.   Plugins should not create arbitrary Configurations, but utilize them for specific purposes and builds.

Configurations actually tie into dependency resolution (with Ivy) and can alter generated pom files.

Configurations should *not* be used to namespace keys for a plugin.  e.g.

```scala
val Config = config("my-plugin")
val pluginKey = SettingKey[String]("plugin-specific-key")
val settings = plugin-key in Config  // DON'T DO THIS!
```

### Using a 'main' task for settings ###

Sometimes you want to define some settings for a particular 'main' task in your plugin.  In this instance, you can scope your settings using the task itself.   For example:

```scala
val plugin-task = TaskKey[Unit]("plugin-awesome-task")
val plugin-settings: Seq[Setting[_]] = Seq(
  sources in Compile in plugin-task <<= ...,
  plugin-task <<= (sources in Compile in plugin-task) map { source => ... }
)
```

