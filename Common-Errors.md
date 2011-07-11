# Common Errors

# Build Compilation Errors

## Confusion between Settings and Tasks.

Settings are evaluated once at load time; tasks are evaluated repeatedly. The code that implements a task is itself stored as a setting, but this is something of an implementation detail, and there are slightly different APIs for configuring the two.

Suppose we define these keys, in `./project/Build.sbt` (For details, see [[Full Configuration]]).

```
val baseSetting = SettingKey[String]("base-setting")
val derivedSetting = SettingKey[String]("derived-setting")
val baseTask = TaskKey[Long]("base-task")
val derivedTask = TaskKey[String]("derived-task")
```

Let's define an initialization for `base-setting` and `base-task`. We will then use these as inputs to other setting and task initializations.
```
baseSetting := "base setting"

baseTask := { System.currentTimeMillis() }
```

One or more settings can be used as inputs to initialize another setting, using the `apply` method.

```
derivedSetting <<= baseSetting.apply(_.toString)

derivedSetting <<= baseSetting(_.toString)

derivedSetting <<= (baseSetting, baseSetting)((a, b) => a.toString + b.toString)
```

Both settings and tasks can be used to initialize a task, using the `map` method.

```
derivedTask <<= baseSetting.map(_.toString)

derivedTask <<= baseTask.map(_.toString)

derivedTask <<= (baseSetting, baseTask).map((a, b) => a.toString + b.toString)
```

But, it is a compile time error to use `map` to initialize a setting:

```
// error: found: Initialize[Task[String]], required: Initialize[String]
derivedSetting <<= baseSetting.map(_.toString),
derivedSetting <<= baseTask.map(_.toString),
derivedSetting <<= (baseSetting, baseTask).map((a, b) => a.toString + b.toString),
```

It is not allowed to use a task as input to a settings initialization with `apply`:

```
// error: value apply is not a member of TaskKey[Long]
derivedSetting <<= baseTask.apply(_.toString)

// error: value apply is not a member of TaskKey[Long]
derivedTask <<= baseTask.apply(_.toString)

// error: value apply is not a member of (sbt.SettingKey[String], sbt.TaskKey[Long])
derivedTask <<= (baseSetting, baseTask).apply((a, b) => a.toString + b.toString)
```

Finally, it is not directly possible to use `apply` to initialize a task.

```
// error: found String, required Task[String]
derivedTask <<= baseSetting.apply(_.toString)
```

# Build Load Errors

## "Reference to uninitialized setting"

Setting initializers are executed in order. If the initialization of a setting depends on other settings that has not been initialized, SBT will stop loading. This can happen using `+=`, `++=`, `<<=`, `<+=`, `<++=`, and `~=`.

In this example, we try to append a library to `libraryDependencies` before it is initialized with an empty sequence.
```
object build extends Build {
  val root = Project(id = "root", base = file("."),
    settings = Seq(
      libraryDependencies += "commons-io" % "commons-io" % "1.4" % "test"
    )
  )
}
```

To correct this, include the default settings, which includes `libraryDependencies := Seq()`.
```
settings = Defaults.defaultSettings ++ Seq(
  libraryDependencies += "commons-io" % "commons-io" % "1.4" % "test"
)
```

A more subtle variation of this error occurs when using scoped settings. The following results in the error `sbt.Init$Uninitialized: Reference to uninitialized setting `.
```
settings = Defaults.defaultSettings ++ Seq(
  libraryDependencies += "commons-io" % "commons-io" % "1.2" % "test",
  fullClasspath ~= (_.filterNot(_.data.name.contains("commons-io")))
)
```

The solution is use the scoped setting as the LHS of the initialization.
```
fullClasspath in Compile ~= (_.filterNot(_.data.name.contains("commons-io")))
```

Generally, all of the update operators can be expressed in terms of `<<=`. The usage of here of the unscoped, and unitialized `fullClasspath` on the RHS of `<<=` that triggers the error.

```
fullClasspath <<= (fullClasspath)(_.filterNot(_.data.name.contains("commons-io")))
```