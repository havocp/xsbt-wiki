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

If your plugin introduces a new concept (even if that concept reuses an existing key), you want your own configuration. For instance, the [LWM](http://software.clapper.org/sbt-lwm/) plugin defines an output directory for the lightweight markup files it transforms. That target directory is scoped in its own configuration, so it is distinct from other output directories. Thus, these two definitions use the same _key_, but they represent distinct _values_. So, in a user's `build.sbt`, we might see:

```scala
target in LWM <<= baseDirectory(_ / "mytarget" / "docs")
target in Compile <<= baseDirectory(_ / "mytarget")
```

In the LWM plugin, this is achieved with an `inConfig` definition:

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

### Playing nice with configurations
Whether you ship with a configuration or not, a plugin should strive to support multiple configurations, including those created by the build user.

Provide unscoped settings in a sequence. This allows the build user to load the family of settings using arbitrary configuration:

```scala
seq(Project.inConfig(Test)(sbtFoo.Plugin.fooSettings0): _*) 
```

Alternatively, one could provide a utility method to load settings in a given configuration:

```scala
seq(fooSettingsInConfig(Test): _*) 
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

#### Provide raw settings and scoped settings
Some tasks that are tied to a particular configuration can be re-used in other configurations.  While you may not see the need immediately in your plugin, some project may and will ask you for the flexibility.   To do so, split your configuration by the configuration axis like so:

```scala
val pluginTask = TaskKey[Unit]("plugin-awesome-task")
val pluginSettings = inConfig(Compile)(basePluginSettings)
val basePluginSettings: Seq[Setting[_]] = Seq(
  pluginTask <<= (sources) map { s => ... }
)
```

The `basePluginSettings` value provides base configuration for the plugin's tasks.  This can be re-used in other configurations if projects require it.   The `pluginSettings` value provides the default `Compile` scoped settings for projects to use directly.  This gives the greatest flexibility in using features provided by a plugin.

#### Using a 'main' task scope for settings

Sometimes you want to define some settings for a particular 'main' task in your plugin.  In this instance, you can scope your settings using the task itself.   For example:

```scala
val pluginTask = TaskKey[Unit]("plugin-awesome-task")
val pluginSettings = inConfig(Compile)(basePluginSettings)
val basePluginSettings: Seq[Setting[_]] = Seq(
  sources in pluginTask <<= ...,
  pluginTask <<= (sources in pluginTask) map { source => ... }
)
```

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