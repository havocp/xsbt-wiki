# Global Settings

## Basic global configuration file

As of 0.10.1, settings that should be applied to all projects can go in a `.sbt` file in `~/.sbt/`.  Plugins that are defined globally in `~/.sbt/plugins` are available to these settings.  For example, to change the default `shellPrompt` for your projects:

`~/.sbt/global.sbt`
```scala
shellPrompt := { state =>
 "sbt (%s)> ".format(Project.extract(state).currentProject.id)
}
```

## Global Settings using a Global Plugin

The previous mechanism for defining global settings still works.  For this approach, one can write a custom plugin in `~/.sbt/plugins` which can change the desired settings.

For example, to change the default `shellPrompt` for your projects, you can create the file `~/.sbt/plugins/ShellPrompt.scala`:

```scala
import sbt._
import Keys._

object ShellPrompt extends Plugin {
  override def settings = Seq(
    shellPrompt := { state =>
      "sbt (%s)> ".format(Project.extract(state).currentProject.id) }
  )
}
```
