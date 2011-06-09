sbt is a build tool for Scala projects that aims to do the basics well. It requires Java 1.6 or later.
test
## Features
* Fairly fast, unintrusive, and easy to set up for simple projects
* [[Configuration|Basic Configuration]], [[customization|Tasks]], and [[extension|Plugins]] are done in Scala
* Accurate recompilation (in theory) is done using information extracted from the compiler
* Continuous compilation and testing with [[triggered execution|Triggered Execution]]
* Supports mixed Scala/[[Java|Java Sources]] projects, packages jars, generates documentation with scaladoc
* Supports [[Testing|testing]] with ScalaCheck, specs, and ScalaTest (JUnit is supported by a plugin)
* Starts the Scala REPL with project classes and dependencies on the classpath
* [[Multi-module and external project|Full Configuration]] support
* Parallel task execution, including parallel test execution
* [[Dependency management support|Library Management]]: inline declarations, external Ivy or Maven configuration files, or manual management

## Getting Started
To get started, read [[Setup]], [[Running]], [[Basic Configuration]], and [[Full Configuration]].  If you are familiar with 0.7.x, please see the [[migration page|Migrating from sbt 0.7.x to 0.10.x]].

The mailing list is at <http://groups.google.com/group/simple-build-tool>. Please use it for questions and comments!

This wiki is editable if you have a github account.  Feel free to make corrections and add documentation.  Use the mailing list if you have questions or comments.
