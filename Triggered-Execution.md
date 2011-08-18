[web plugin]: https://github.com/siasia/xsbt-web-plugin

# Triggered Execution

You can make a command run when certain files change by prefixing the command with `~`.  Monitoring is terminated when `enter` is pressed.  This triggered execution is configured by the `watch` setting, but typically the basic settings `watch-sources` and `poll-interval` are modified.

* `watch-sources` defines the files for a single project that are monitored for changes.  By default, a project watches resources and Scala and Java sources.
* `watch-transitive-sources` then combines the `watch-sources` for the current project and all execution and classpath dependencies (see [[Full Configuration]] for details on inter-project dependencies).
* `poll-interval` selects the interval between polling for changes in milliseconds.  The default value is `500 ms`.

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

If you use `jetty-run` from the [web plugin], you can automatically build and reload your web application on changes to your source:

```scala
> jetty-run
> ~ prepare-webapp
```

This will first start Jetty and then trigger execution of the action that builds the web application provided to Jetty.  The `jetty-run` action will pick up changes and redeploy.