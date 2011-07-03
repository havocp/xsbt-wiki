[State]: http://harrah.github.com/xsbt/latest/api/sbt/State$.html
[xsbti.ComponentProvider]: http://harrah.github.com/xsbt/latest/api/xsbti/ComponentProvider.html

# Specialized Tasks

This page discusses how to perform some tasks that are less common.

## Augment sbt's classpath dynamically

It is possible to register additional jars that will be placed on sbt's classpath (since version 0.10.1).
Through [State], it is possible to obtain a [xsbti.ComponentProvider], which manages application components.
Components are groups of files in the `project/boot/` directory and, in this case, the application is sbt.
In addition to the base classpath, components in the "extra" component are included on sbt's classpath.

(Note: the additional components on an application's classpath are declared by the `components` property in the `[main]` section of the launcher configuration file `boot.properties`.)

Because these components are added to the `project/boot/` directory and `project/boot/` may be read-only, this can fail.
In this case, the user has generally intentionally set sbt up this way, so error recovery is not typically necessary (just a short error message explaining the situation.)

### Example

The following code can be used where a `State => State` is required, such as in the `onLoad` setting or in a [[command|Commands]].
It adds some files to the "extra" component and reloads sbt if they were not already added.
Note that reloading will drop the user's session state.

```scala
def augment(extra: Seq[File])(s: State): State =
{
    // Get the component provider
  val cs: xsbti.ComponentProvider = s.configuration.provider.components()

    // Adds the files in 'extra' to the "extra" component
    //   under an exclusive machine-wide lock.
    //   The returned value is 'true' if files were actually copied and 'false'
    //   if the target files already exists (based on name only).
  val copied: Boolean = s.locked(cs.lockFile, cs.addToComponent("extra", extra.toArray))

    // If files were copied, reload so that we use the new classpath.
  if(copied) s.reload else s
}
```

