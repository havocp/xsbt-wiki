[patterns]: http://ant.apache.org/ivy/history/latest-milestone/concept.html#patterns
[Patterns API]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/Patterns$object.html
[Ivy filesystem]: http://ant.apache.org/ivy/history/latest-milestone/resolver/filesystem.html (Ivy)
[filesystem factory]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/Resolver$object.file$object.html
[FileRepository API]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/FileRepository.html
[Ivy sftp]: http://ant.apache.org/ivy/history/latest-milestone/resolver/sftp.html
[sftp factory]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/Resolver$object.Define.html
[SftpRepository API]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/SftpRepository.html
[Ivy ssh]: http://ant.apache.org/ivy/history/latest-milestone/resolver/ssh.html
[ssh factory]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/Resolver$object.Define.html
[SshRepository API]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/SshRepository.html
[Ivy url]: http://ant.apache.org/ivy/history/latest-milestone/resolver/url.html
[url factory]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/Resolver$object.url$object.html
[URLRepository API]: http://simple-build-tool.googlecode.com/svn/artifacts/latest/api/sbt/URLRepository.html

# Resolvers

Maven-style repositories are defined like:
```scala
resolvers += "Scala-Tools Maven2 Snapshots Repository" at "http://scala-tools.org/repo-snapshots"
```
and use the Maven2 layout.  This page describes how to define other types of repositories. 

# Predefined

A few predefined repositories are available and are listed below

* `DefaultMavenRepository`
 This is the main Maven repository at [[http://repo1.maven.org/maven2/]] and is included by default
* `ScalaToolsReleases`
 This is the releases repository at [[http://scala-tools.org/repo-releases/]] and is included by default
* `ScalaToolsSnapshots`
 This is the snapshots repository at [[http://scala-tools.org/repo-snapshots/]]
* `JavaNet1Repository`
 This is the Maven 1 repository at [[http://download.java.net/maven/1/]]

For example, to use the `java.net` repository, use the following setting in your build definition:
```scala
resolvers += JavaNet1Repository
```

# Custom

`sbt` provides an interface to the repository types available in Ivy: file, URL, SSH, and SFTP.  A key feature of repositories in Ivy is using [patterns] to configure repositories.

Construct a repository definition using the factory in `sbt.Resolver` for the desired type.  This factory creates a `Repository` object that can be further configured.  The following table contains links to the Ivy documentation for the repository type and the API documentation for the factory and repository class.  The SSH and SFTP repositories are configured identically except for the name of the factory.  Use `Resolver.ssh` for SSH and `Resolver.sftp` for SFTP.

|| Type || Factory || Ivy Docs || Factory API || Repository Class API ||
|| Filesystem || `Resolver.file` || [Ivy filesystem] || [filesystem factory] || [FileRepository API] ||
|| SFTP || `Resolver.sftp` || [Ivy sftp] || [sftp factory] || [SftpRepository API] ||
|| SSH || `Resolver.ssh` || [Ivy ssh] || [ssh factory] || [SshRepository API] ||
||  URL || `Resolver.url` || [Ivy url] || [url factory] || [URLRepository API] ||

## Basic Examples

These are basic examples that use the default Maven-style repository layout.

### Filesystem

Define a filesystem repository in the `test` directory  of the current working directory and declare that publishing to this repository must be atomic.
```scala
resolvers += Resolver.file("my-test-repo", new java.io.File("test")) transactional()
```

### URL

Define a URL repository at .`"http://example.org/repo-releases/"`.
```scala
{
  def url = new java.net.URL("http://example.org/repo-releases/")
  resolvers += Resolver.url("my-test-repo", url)
}
```

To specify an Ivy repository, use:
```
resolvers += Resolver.url("my-test-repo", url)(Resolver.ivyStylePatterns)
```
or customize the layout pattern described in the Custom Layout section below.

### SFTP and SSH Repositories

The following defines a repository that is served by SFTP from host `"example.org"`:
```scala
resolvers += Resolver.sftp("my-sftp-repo", "example.org")
```

To explicitly specify the port:
```scala
resolvers += Resolver.sftp("my-sftp-repo", "example.org", 22)
```

To specify a base path:
```scala
resolvers += Resolver.sftp("my-sftp-repo", "example.org", "maven2/repo-releases/")
```

Authentication for the repositories returned by `sftp` and `ssh` can be configured by the `as` methods.

To use password authentication:
```scala
resolvers += Resolver.ssh("my-ssh-repo", "example.org") as("user", "password")
```

or to be prompted for the password:
```scala
resolvers += Resolver.ssh("my-ssh-repo", "example.org") as("user")
```

To use key authentication:
```scala
resolvers += {
  val keyFile: File = ...
  Resolver.ssh("my-ssh-repo", "example.org") as("user", keyFile, "keyFilePassword")
}
```

or if no keyfile password is required or if you want to be prompted for it:
```scala
resolvers += Resolver.ssh("my-ssh-repo", "example.org") as("user", keyFile)
```

To specify the permissions used when publishing to the server:
```scala
resolvers += Resolver.ssh("my-ssh-repo", "example.org") withPermissions("0644")
```

This is a chmod-like mode specification.

## Custom Layout

These examples specify custom repository layouts using patterns.  The factory methods accept an `Patterns` instance that defines the patterns to use.  The patterns are first resolved against the base file or URL.  The default patterns give the default Maven-style layout.  Provide a different Patterns object to use a different layout.  For example:
```scala
resolvers += Resolver.url("my-test-repo", url)( Patterns("[organisation]/[module]/[revision]/[artifact].[ext]") )
```

You can specify multiple patterns or patterns for the metadata and artifacts separately.  You can also specify whether the repository should be Maven compatible (as defined by Ivy).  See the [patterns API] for the methods to use.

For filesystem and URL repositories, you can specify absolute patterns by omitting the base URL, passing an empty `Patterns` instance, and using `ivys` and `artifacts`:
```scala
resolvers += Resolver.url("my-test-repo") artifacts
        "http://example.org/[organisation]/[module]/[revision]/[artifact].[ext]"
```