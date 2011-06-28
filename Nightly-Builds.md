[sbt-launch]: http://repo.typesafe.com/typesafe/ivy-snapshots/org.scala-tools.sbt/sbt-launch/

# Nightly Builds

Nightly builds are currently being published to <http://repo.typesafe.com/typesafe/ivy-snapshots/>.

To use a nightly build, follow the instructions for normal [[Setup]], except:

1. Download the launcher jar from one of the subdirectories of [sbt-launch].  They should be listed in chronological order, so the most recent one will be last.
2. Call your script something like `sbt-nightly` to retain access to a stable `sbt` launcher.
3. The version number is the name of the subdirectory and is of the form `0.10.x-yyyyMMdd-HHmmss`.  Use this in a `build.properties`.

Related to the third point, remember that an `sbt.version` setting in `<build-base>/project/build.properties` determines the version of sbt to use in a project.  If it is not present, the default version associated with the launcher is used.  This means that you must set `sbt.version=yyyyMMdd-HHmmss` in an existing `<build-base>/project/build.properties`.  You can verify the right version of sbt is being used to build a project by running `sbt-version`.

A nightly launcher jar should be able to launch previous stable 0.10.x versions of sbt (it is a bug otherwise).

However, to reduce problems, it is recommended to not use a launcher jar for one nightly version to launch a different nightly version of sbt.  That is, if you have launcher `0.10.1-20110628-110226`, only specify `0.10.0` and `0.10.1-20110628-110226` in a `project/build.properties` file.