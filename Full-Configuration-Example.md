Small example showing a project with two subprojects and establishing the resolvers and dependencies for each.

```scala
import sbt._
import Keys._

object MyBuild extends Build {

  lazy val projects = Seq (myapp, common, server)
  
  val myapp = Project ("myapp", file(".")) dependsOn (common, server)
  
  val sunrepo = "Sun Maven2 Repo" at "http://download.java.net/maven/2"
  val sunrepoGF = "Sun GF Maven2 Repo" at "http://download.java.net/maven/glassfish" 
  val oraclerepo = "Oracle Maven2 Repo" at "http://download.oracle.com/maven"     

  val oracleResolvers = Seq (sunrepo, sunrepoGF, oraclerepo)

  val jacksonVer = "1.7.2"  
  val logbackVersion = "0.9.16"
  val logbackcore    = "ch.qos.logback" % "logback-core"     % logbackVersion
  val logbackclassic = "ch.qos.logback" % "logback-classic"  % logbackVersion  
  val jacksonjson    = "org.codehaus.jackson" % "jackson-core-lgpl" % jacksonVer
  
  val grizzlyVersion  = "1.9.19"
  val grizzlyframwork = "com.sun.grizzly" % "grizzly-framework"  % grizzlyVersion
  val grizzlyhttp     = "com.sun.grizzly" % "grizzly-http"       % grizzlyVersion
  val grizzlyrcm      = "com.sun.grizzly" % "grizzly-rcm"        % grizzlyVersion
  val grizzlyutils    = "com.sun.grizzly" % "grizzly-utils"      % grizzlyVersion
  val grizzlyportunif = "com.sun.grizzly" % "grizzly-portunif"   % grizzlyVersion
  val sleepycat       = "com.sleepycat" % "je" % "4.0.92"

  val commonDeps = Seq (logbackcore, logbackclassic, jacksonjson)

  val serverDeps = Seq (grizzlyframwork, grizzlyhttp, grizzlyrcm, grizzlyutils, grizzlyportunif, sleepycat)

  lazy val common = Project ("common", file ("common")) settings (libraryDependencies := commonDeps, 
									scalaVersion := "2.9.0-1")

  lazy val server = Project ("server", file ("server")) settings (resolvers := oracleResolvers, 
									libraryDependencies := serverDeps, 
									scalaVersion := "2.9.0-1") dependsOn (common)
}
```