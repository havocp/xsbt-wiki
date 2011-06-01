Small example showing a project with two subprojects and establishing the resolvers and dependencies for each.

```scala
import sbt._

import Keys._

// Shell prompt which show the current project and git branch
// git magic from Daniel Sobral
object ShellPrompt {
 
  object devnull extends ProcessLogger {
    def info (s: => String) {}
    def error (s: => String) { }
    def buffer[T] (f: => T): T = f
  }
  
  val current = """\*\s+(\w+)""".r
  
  def gitBranches = ("git branch --no-color" lines_! devnull mkString)
  
  val buildShellPrompt = { 
    (state: State) => {
      val currBranch = current findFirstMatchIn gitBranches map (_.group(1)) getOrElse "-"
      val currProject = Project.extract (state).currentProject.id
      "%s:%s> ".format (currProject, currBranch)
    }
  }
 
}

object Resolvers {

  val sunrepo    = "Sun Maven2 Repo" at "http://download.java.net/maven/2"
  val sunrepoGF  = "Sun GF Maven2 Repo" at "http://download.java.net/maven/glassfish" 
  val oraclerepo = "Oracle Maven2 Repo" at "http://download.oracle.com/maven"

  val oracleResolvers = Seq (sunrepo, sunrepoGF, oraclerepo)
  
}

object Dependencies {

  val jacksonVersion = "1.7.2"  
  val logbackVersion = "0.9.16"

  val logbackcore    = "ch.qos.logback" % "logback-core"     % logbackVersion
  val logbackclassic = "ch.qos.logback" % "logback-classic"  % logbackVersion  
  val jacksonjson    = "org.codehaus.jackson" % "jackson-core-lgpl" % jacksonVersion
  
  val grizzlyVersion  = "1.9.19"
  val grizzlyframwork = "com.sun.grizzly" % "grizzly-framework"  % grizzlyVersion
  val grizzlyhttp     = "com.sun.grizzly" % "grizzly-http"       % grizzlyVersion
  val grizzlyrcm      = "com.sun.grizzly" % "grizzly-rcm"        % grizzlyVersion
  val grizzlyutils    = "com.sun.grizzly" % "grizzly-utils"      % grizzlyVersion
  val grizzlyportunif = "com.sun.grizzly" % "grizzly-portunif"   % grizzlyVersion
  val sleepycat       = "com.sleepycat"   % "je"                 % "4.0.92"

  val apachenet   = "commons-net"   % "commons-net"   % "2.0"
  val apachecodec = "commons-codec" % "commons-codec" % "1.4"

  val scalatest = "org.scalatest" % "scalatest_2.9.0" % "1.4.1" % "test"
}


object CDAP2Build extends Build {

  val buildOrganization = "odp"
  val buildVersion      = "2.0.28"
  val buildScalaVersion = "2.9.0-1"

  val buildShellPrompt = ShellPrompt.buildShellPrompt
  
  import Resolvers._
  import Dependencies._

  // Sub-project specific dependencies
  val commonDeps = Seq (logbackcore, logbackclassic, jacksonjson)

  val serverDeps = Seq (grizzlyframwork, grizzlyhttp, grizzlyrcm, grizzlyutils, grizzlyportunif, sleepycat, scalatest)

  val pricingDeps = Seq (apachenet, apachecodec)


  // Projects
  lazy val projects = Seq (cdap2, pricing, common, compact, server, pricing_service)
  
  lazy val cdap2 = Project ("cdap2", file (".")) aggregate (common, server, compact, 
							    pricing, pricing_service) settings (shellPrompt := buildShellPrompt)

  lazy val common = Project ("common", file ("cdap2-common")) settings (libraryDependencies := commonDeps, 
									organization := buildOrganization,
									version := buildVersion, 
									shellPrompt := buildShellPrompt,
									scalaVersion := buildScalaVersion)

  lazy val server = Project ("server", file ("cdap2-server")) settings (resolvers := oracleResolvers, 
									organization := buildOrganization,
									version := buildVersion, 
									libraryDependencies := serverDeps, 
									shellPrompt := buildShellPrompt,
									scalaVersion := buildScalaVersion) dependsOn (common)

  lazy val pricing = Project ("pricing", file ("cdap2-pricing")) settings (scalaVersion := buildScalaVersion,
									   version := buildVersion, 
									   organization := buildOrganization,
									   shellPrompt := buildShellPrompt,
									   libraryDependencies := pricingDeps) dependsOn (common, compact, server)

  lazy val pricing_service = Project ("pricing-service", file ("cdap2-pricing-service")) settings (scalaVersion := buildScalaVersion,
												   organization := buildOrganization,
												   shellPrompt := buildShellPrompt,
												   version := buildVersion) dependsOn (pricing, server)

  lazy val compact = Project ("compact", file ("compact-hashmap")) settings (scalaVersion := buildScalaVersion,
									     organization := buildOrganization,
									     shellPrompt := buildShellPrompt,
									     version := buildVersion)
}

```