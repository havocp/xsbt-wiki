[Path]: http://harrah.github.com/xsbt/latest/api/sbt/Path$.html
[PathFinder]: http://harrah.github.com/xsbt/latest/api/sbt/PathFinder.html

# Mapping Files

Tasks like `package`, `packageSrc`, and `packageDoc` accept mappings from an input file to the path to use in the resulting artifact (jar).  There are some methods on [PathFinder] and [Path] that can be useful for constructing the `Seq[(File, String)]`.

A common way of making this sequence is to start with a `PathFinder` or `Seq[File]` (which is implicitly convertible to `PathFinder`) and then call the `x` method.  See the [PathFinder] API for details, but essentially this method accepts a function `File => Option[String]` that is used to generate mappings.

## Relative to a directory

The `Path.relativeTo` method is used to map a `File` to its path relative to a base directory or directories.  The `relativeTo` method accepts a base directory or sequence of base directories to relativize an input file against.  The first directory that is an ancestor of the file is used in the case of a sequence of base directories.

For example:

```scala
  import Path.relativeTo
val files: Seq[File] = file("/a/b/C.scala") :: Nil
val baseDirectories: Seq[File] = file("/a") :: Nil
val mappings: Seq[(File,String)] = files x relativeTo(baseDirectories)

val expected = (file("/a/b/C.scala") -> "b/C.scala") ) :: Nil
assert( mappings == expected )
```

## Rebase

The `Path.rebase` method relativizes an input file against one or more base directories (the first argument) and then prepends a base String (the second argument) to the result.  As with `relativeTo`, the first base directory that is an ancestor of the input file is used in the case of multiple base directories.

```scala
  import Path.rebase
val files: Seq[File] = file("/a/b/C.scala") :: Nil
val baseDirectories: Seq[File] = file("/a") :: Nil
val mappings: Seq[(File,String)] = files x rebase(baseDirectories, "pre/")

val expected = (file("/a/b/C.scala") -> "pre/b/C.scala") ) :: Nil
assert( mappings == expected )
```

## Flatten

The `Path.flat` method provides a function that maps a file to the last component of the path (its name).  For example:

```scala
  import Path.flat
val files: Seq[File] = file("/a/b/C.scala") :: Nil
val mappings: Seq[(File,String)] = files x flat

val expected = (file("/a/b/C.scala") -> "C.scala") ) :: Nil
assert( mappings == expected )
```

## Alternatives

To try to apply several alternative mappings for a file, use `|`, which is implicitly added to a function of type `A => Option[B]`.  For example, to try to relativize a file against some base directories but fall back to flattening:

```scala
  import Path.relativeTo
val files: Seq[File] = file("/a/b/C.scala") :: file("/zzz/D.scala") :: Nil
val baseDirectories: Seq[File] = file("/a") :: Nil
val mappings: Seq[(File,String)] = files x ( relativeTo(baseDirectories) | flat )

val expected = 
  (file("/a/b/C.scala") -> "b/C.scala") ) ::
  (file("/zzz/D.scala") -> "D.scala") ) ::
  Nil
assert( mappings == expected )
```