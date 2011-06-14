As of 0.10.0, there is [not yet](https://github.com/harrah/xsbt/issues/52) a method for simply declaring global settings.  Instead, one can write a custom plugin in `~/.sbt/plugins` which can change the desired settings.

For example, to change the default `shellPrompt` for your SBT project, you can create the file `~/.sbt/plugins/ShellPrompt.scala`:

    import sbt._
    import Keys._
    
    object ShellPrompt extends Plugin {
      override def settings = Seq(
        shellPrompt := { state =>
          "sbt (%s)> ".format(Project.extract(state).currentProject.id) }
      )
    }
    