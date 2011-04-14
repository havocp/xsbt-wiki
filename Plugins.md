# Plugins (Draft)

# Introduction

A plugin is essentially a way to use external code in a build definition.
A plugin can be a library used to implement a task.  For example, you might use [Knockoff](http://tristanhunt.com/projects/knockoff/) to write a markdown processing task.
A plugin can define a sequence of sbt Settings that are automatically added to all projects or that are explicitly declared for selected projects.
For example, a plugin might add a 'proguard' task and associated (overridable) settings.
Because [[Commands]] can be added with the `commands` setting, a plugin can also fulfill the role that processors did in 0.7.x.

# By Description

A plugin definition is a project in `<main-project>/project/plugins/`.
This project's classpath is the classpath used for build definitions in `<main-project>/project/` and any `.sbt` files.  It is also used for the `eval` and `set` commands.

Specifically,

1. Managed dependencies declared by the `project/plugins/` project are retrieved and put on the build definition classpath.
2. Unmanaged dependencies in `project/plugins/lib/` are available to the build definition.
3. Sources in the `project/plugins/` project are compiled and put on the build definition claspath.
4. Project dependencies can be declared in `project/plugins/project/Build.scala` and will be available to the build definition.

The build definition classpath is searched for `sbt/sbt.plugins` descriptor files containing the names of Plugin implementations.
A Plugin is a module that defines settings to automatically inject to projects.
Additionally, all Plugin modules are wildcard imported for the `eval` and `set` commands and `.sbt` files.
A Plugin implementation is not required to produce a plugin, however.
It is a convenience for plugin consumers and because of the automatic nature, it is not always appropriate.

The `reload plugins` command changes the current build to `<current-build>/project/plugins/`.
This allows manipulating the plugin definition project like a normal project.
`reload return` changes back to the original build.
Any session settings for the plugin definition project that have not been saved are dropped.

### Global plugins

In sbt 0.7.x, a processor was a way to add new commands to sbt and distribute them to users.  A key feature was the ability to have per-user processors so that once declared, it could be used in all projects for that user.  In sbt 0.9, plugins and processors are unified.  Specifically, a plugin can add commands and plugins can be declared globally for a user.

The `~/.sbt/plugins/` directory is treated as a global plugin definition project.  It is a normal sbt project whose classpath is available to all sbt project definitions for that user as described above for per-project plugins.

# By Example

## Using a library in a build definition
As an example, we'll add the Grizzled Scala library as a plugin.  Although this does not provide sbt-specific functionality, it demonstrates how to declare plugins.

### 1a) Manually managed

1. Download the jar manually from [[http://www.scala-tools.org/repo-releases/org/clapper/grizzled-scala_2.8.1/1.0.4/grizzled-scala_2.8.1-1.0.4.jar]]
2. Put it in `project/plugins/lib/`

### 1b) Automatically managed: direct editing approach

Edit `project/plugins/build.sbt` to contain:
```scala
set libraryDependencies += "org.clapper" %% "grizzled-scala" % "1.0.4"
```

If sbt is running, do `reload`.

### 1c) Automatically managed: command line approach

We can change to the plugins project in `project/plugins/` using `reload plugins`.
```console
$ xsbt
> reload plugins
[info] Set current project to default (in build file:/Users/harrah/demo2/project/plugins/)
>
```

Then, we can add dependencies like usual and save them to `project/plugins/build.sbt`.
It is useful, but not required, to run `update` to verify that the dependencies are correct.
```console
> set libraryDependencies += "org.clapper" %% "grizzled-scala" % "1.0.4"
...
> update
...
> session save
...
```

To switch back to the main project:
```console
> reload return
[info] Set current project to root (in build file:/Users/harrah/demo2/)
```

### 1d) Project dependency

This variant shows how to use the external project support in sbt 0.9 to declare a source dependency on a plugin.
This means that the plugin will be built from source and used on the classpath.
This is example is currently only for show, since the plugin needs to build with sbt 0.9 for this to work.

Edit `project/plugins/project/Build.scala`
```scala
import sbt._
object PluginDef extends Build {
	lazy val projects = Seq(root)
	lazy val root = Project("plugins", file(".")) dependsOn(grizzled)
	lazy val grizzled = uri("git://github.com/bmc/grizzled-scala.git")
}
```
If sbt is running, run `reload`.

Note that this approach can be useful used when developing a plugin.
A project that uses the plugin will rebuild the plugin on `reload`.
This saves the intermediate steps of `publish-local` and `clean-plugins` required in 0.7.
It can also be used to work with the development version of a plugin from its repository.

### 2) Use the library

Grizzled Scala is ready to be used in build definitions.
This includes the `eval` and `set` commands and `.sbt` and `project/*.scala` files. 

```console
> eval grizzled.sys.os
```


In a `build.sbt` file:
```scala
import grizzled.sys._
import OperatingSystem._

libraryDependencies ++=
	if(os ==Windows)
		("org.example" % "windows-only" % "1.0") :: Nil
	else
		Nil
```

# Creating a plugin

## Introduction

A minimal plugin is a Scala library that is built against the version of Scala that sbt runs (currently, 2.8.1) or a Java library.
Nothing special needs to be done for this type of library, as shown in the previous section.
A more typical plugin will provide sbt tasks, commands, or settings.  This kind of plugin may provide these settings automatically or make them available for the user to explicitly integrate.

## Description

To make a plugin, create a project and configure `sbtPlugin` to `true`.
Then, write the plugin code and publish your project to a repository.
The plugin can be used as described in the previous section.

A plugin can implement `sbt.Plugin`.
The contents of a Plugin singleton, declared like `object MyPlugin extends Plugin`, are wildcard imported in `set`, `eval`, and `.sbt` files.
Typically, this is used to provide new keys (SettingKey, TaskKey, or InputKey) or core methods without requiring an import or qualification.
In addition, the `settings` member of the `Plugin` is automatically appended to each project's settings.
This allows a plugin to automatically provide new functionality or new defaults.
One main use of this feature is to globally add commands, like a processor in sbt 0.7.x.
These features should be used judiciously because the automatic activation removes control from the build author (the user of the plugin).

## Example Plugin

An example of a typical plugin:

`build.sbt`:
```scala
sbtPlugin := true

name := "example-plugin"

organization := "org.example"
```

`MyPlugin.scala`:
```scala
import sbt._
object MyPlugin extends Plugin
{
	// configuration points, like the built in `version`, `libraryDependencies`, or `compile`
	// by implementing Plugin, these are automatically imported in a user's `build.sbt`
	val newTask = TaskKey[Unit]("new-task")
	val newSetting = SettingKey[String]("new-setting")

	// a group of settings ready to be added to a Project
	// to automatically add them, do 
	val newSettings = Seq(
		newSetting := "test",
		newTask <<= newSetting map { str => println(str) }
	)

	// alternatively, by overriding `settings`, they could be automatically added to a Project
	// override val settings = Seq(...)
}
```

## Usage example

A light build definition that uses the plugin might look like:
```scala
seq( MyPlugin.newSettings : _*)

newSetting := "light"
```

A full build definition that uses this plugin might look like:
```scala
object MyBuild extends Build
{
	lazy val projects = Seq(root)
	lazy val root = Project("root", file(".")) settings( MyPlugin.newSettings : _*) settings(
		newSetting := "full"
	)
}
```

Individual settings could be defined in `MyBuild.scala` above or in a `build.sbt` file:
```scala
newSettings := "overridden"
```

## Example command plugin
A basic plugin that adds commands looks like:

`build.sbt`
```scala
sbtPlugin := true

name := "example-plugin"

organization := "org.example"
```

`MyPlugin.scala`
```scala
import sbt._
import Keys._
object MyPlugin extends Plugin
{
  override lazy val settings = Seq(commands += myCommand)

  lazy val myCommand = 
    Command.command("hello") { (state: State) =>
      println("Hi!")
      state
    }
}
```

This example demonstrates how to take a Command (here, `myCommand`) and distribute it in a plugin.  Note that multiple commands can be included in one plugin (for example, use `commands ++= Seq(a,b)`).  See [[Commands]] for defining more useful commands, including ones that accept arguments and affect the execution state.

## Global plugins example
The simplest global plugin definition is declaring a library or plugin in `~/.sbt/plugins/build.sbt`:
```scala
libraryDependencies += "org.example" %% "my-plugin" % "0.1"
```
This plugin will be available for every sbt project for the current user.

In addition:

1. Jars may be placed directly in `~/.sbt/plugins/lib/` and will be available to every build definition for the current user.
2. Dependencies on plugins built from source may be declared in ~/.sbt/plugins/project/Build.scala` as described at [[FullConfiguration]].
3. A Plugin may be directly defined in Scala source files in `~/.sbt/plugins/`, such as `~/.sbt/plugins/MyPlugin.scala`.  `~/.sbt/plugins/build.sbt` should contain `sbtPlugin := true`.  This can be used for quicker turnaround when developing a plugin initially:
    1. Edit the global plugin code
    2. `reload` the project you want to use the modified plugin in
    3.  sbt will rebuild the plugin and use it for the project.  Additionally, the plugin will be available in other projects on the machine without recompiling again.
This approach skips the overhead of `publish-local` and cleaning the plugins directory of the project using the plugin.

These are all consequences of `~/.sbt/plugins/` being a standard project whose classpath is added to every sbt project's build definition.