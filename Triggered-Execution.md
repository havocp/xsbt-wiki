# Triggered Execution

You can make a command run when certain files change by prefixing the command with `~`.  This triggered execution is configured by the `watch` setting, but typically the basic settings `watch-sources` and `poll-interval` are modified.  `watch-sources` defines the files for a single project that are monitored for changes.  By default, a project watches resources and Scala and Java sources.  `watch-transitive-sources` then combines the `watch-sources` for the current project and all execution and classpath dependencies (see [[Full Configuration]] for details on inter-project dependencies).  Monitoring is terminated by default when `enter` is pressed.  The interval between polling is configured by the `poll-interval` setting, which is in milliseconds and is `500 ms` by default.

Some example usages are described below.

# Compile

The original use-case was continuous compilation:
```scala
> ~ test-compile

> ~ compile
```

# Testing

You can use the triggered execution feature to run any command or task.  One use is for test driven development, as suggested by Erick on the mailing list.

The following will poll for changes to your source code (main or test) and run `test-only` for the specified test.
```scala
> ~ test-only example.TestA
```

## Web Applications

**Web support not implemented**

If you use `jetty-run`, you can automatically build and reload your web application on changes to your source:

```scala
> jetty-run
> ~ prepare-webapp
```
This will first start Jetty and then trigger execution of the action that builds the web application provided to Jetty.  The `jetty-run` action will pick up changes and redeploy.