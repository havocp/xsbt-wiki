# Testing (In progress)

## Disable Parallel Execution of Tests

```scala
parallelExecution in Test := false
```

`Test` can be replaced with `IntegrationTest` to only execute integration tests serially.

## Example: Integration Tests

```scala
  import sbt._
  import Keys._

object B extends Build
{
  lazy val projects = Seq(root)
  lazy val root =
    Project("root", file("."))
      .configs( IntegrationTest )
      .settings( Defaults.itSettings : _*)
      .settings( libraryDependencies += specs )

  lazy val specs = "org.scala-tools.testing" %% "specs" % "1.6.7.2" % "it,test" intransitive()
}
```

## Example: Custom test configuration

```scala
  import sbt._
  import Keys._

object B extends Build
{
  lazy val projects = Seq(root)
  lazy val root =
    Project("root", file("."))
      .configs( FunTest )
      .settings( inConfig(FunTest)(Defaults.testSettings) : _*)
      .settings( libraryDependencies += specs )

  lazy val FunTest = config("fun") extend(Test)
  lazy val specs = "org.scala-tools.testing" %% "specs" % "1.6.7.2" % "fun" intransitive()
}
```