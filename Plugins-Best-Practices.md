# Plugins Best Practices

_This page is intended primarily for SBT plugin authors._

A plugin developer should strive for consistency and ease of use. Specifically:

* Plugins should play well with other plugins. Avoiding namespace clashes (in both SBT and Scala) is paramount.
* Plugins should follow consistent conventions. The experiences of an SBT _user_ should be consistent, no matter
  what plugins are pulled in.

Here are some current plugin best practices. **NOTE:** Best practices are evolving, so check back frequently.

## `settings` are for commands

SBT will automatically load your plugin's `settings` into the build. Overriding `val settings` should only be done by plugins providing commands. Regular plugins defining tasks and settings should provide a sequence named after the plugin like so:

```scala
val obfuscateSettings = Seq(...)
```

This allows build user to choose which subproject the plugin would be used. See later section for how the settings should be scoped.

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
  val fooStylesheet = SettingKey[File]("foo-stylesheet")
}
```

In this approach, every `val` starts with `foo`. A user of the plugin would refer to the settings like this:

```
fooStylesheet <<= ...
```

### Use a nested object

```scala
object FooPlugin extends sbt.Plugin {
  object foo {
    val stylesheet = SettingKey[File]("foo-stylesheet")
  }
}
```

In this approach, all non-common settings are in a nested object. A user of the plugin would refer to the settings like this:

```
foo.stylesheet <<= ...
```

## Configuration Advice
### When to define your own configuration

If your plugin introduces a new concept (even if that concept reuses an existing key), you want your own configuration. For instance, suppose you've built a plugin that produces PDF files from some kind of markup, and your plugin defines a target directory to receive the resulting PDFs. That target directory is scoped in its own configuration, so it is distinct from other target directories. Thus, these two definitions use the same _key_, but they represent distinct _values_. So, in a user's `build.sbt`, we might see:

```scala
target in PDFPlugin <<= baseDirectory(_ / "mytarget" / "pdf")
target in Compile <<= baseDirectory(_ / "mytarget")
```

In the PDF plugin, this is achieved with an `inConfig` definition:

```scala
val settings: Seq[sbt.Project.Setting[_]] = inConfig(LWM)(Seq(
  target <<= baseDirectory(_ / "target" / "docs") # the default value
))
```

### When _not_ to define your own configuration.

If you're merely adding to existing definitions, don't define your own configuration. Instead, reuse an existing one _or_ scope by the main task (see below).

```scala
val akka = config("akka")  // This isn't needed.
val akkaStartCluster = TaskKey[Unit]("akka-start-cluster")

target in akkaStartCluster <<= ... // This is ok.
akkaStartCluster in akka <<= ...   // BAD.  No need for a Config for plugin-specific task.
```

### Configuration Cat says "Configuration is for configuration"

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

### Playing nice with configurations
Whether you ship with a configuration or not, a plugin should strive to support multiple configurations, including those created by the build user. Some tasks that are tied to a particular configuration can be re-used in other configurations.  While you may not see the need immediately in your plugin, some project may and will ask you for the flexibility.

#### Provide raw settings and configured settings
Split your settings by the configuration axis like so:

```scala
val obfuscate = TaskKey[Seq[File]]("obfuscate")
val obfuscateSettings = inConfig(Compile)(baseObfuscateSettings)
val baseObfuscateSettings: Seq[Setting[_]] = Seq(
  obfuscate <<= (sources in obfuscate) map { s => ... },
  sources in obfuscate <<= (sources).identity
)
```

The `baseObfuscateSettings` value provides base configuration for the plugin's tasks.  This can be re-used in other configurations if projects require it.   The `obfuscateSettings` value provides the default `Compile` scoped settings for projects to use directly. This gives the greatest flexibility in using features provided by a plugin. Here's how the raw settings may be reused:

```scala
seq(Project.inConfig(Test)(sbtObfuscate.Plugin.baseObfuscateSettings): _*) 
```

Alternatively, one could provide a utility method to load settings in a given configuration:

```scala
def obfuscateSettingsIn(c: Configuration): Seq[Project.Setting[_]] =
  inConfig(c)(baseObfuscateSettings)
```

This could be used as follows:

```scala
seq(obfuscateSettingsIn(Test): _*) 
```

#### Using a 'main' task scope for settings

Sometimes you want to define some settings for a particular 'main' task in your plugin.  In this instance, you can scope your settings using the task itself.

```scala
val obfuscate = TaskKey[Seq[File]]("obfuscate")
val obfuscateSettings = inConfig(Compile)(baseObfuscateSettings)
val baseObfuscateSettings: Seq[Setting[_]] = Seq(
  obfuscate <<= (sources in obfuscate) map { s => ... },
  sources in obfuscate <<= (sources).identity
)
```

In the above example, `sources in obfuscate` is scoped under the main task, `obfuscate`.

## Mucking with Global build state

There may be times when you need to muck with global build state.  The general rule is *be careful what you touch*.  

First, make sure your user do not include global build configuration in *every* project but rather in the build itself.   e.g.

```scala
object MyBuild extends Build {
  override lazy val settings = super.settings ++ MyPlugin.globalSettings
  val main = project(file("."), "root") settings(MyPlugin.globalSettings:_*) // BAD!
}
```

Global settings should *not* be placed into a `build.sbt` file.

When overriding global settings, care should be taken to ensure previous settings from other plugins are not ignored.   e.g. when creating a new `onLoad` handler, ensure that the previous `onLoad` handler is not removed.

```scala
object MyPlugin extends Plugin {
  val globalSettigns: Seq[Setting[_]] = Seq(
    onLoad in Global <<= onLoad in Global apply (_ andThen { state => 
       ... return new state ...
    })
  )
}
```