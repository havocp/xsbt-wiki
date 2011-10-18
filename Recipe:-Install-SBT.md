## Problem
You want to install sbt.

## Solution
### Unix, including Mac OS
1. Download the [launcher jar](http://typesafe.artifactoryonline.com/typesafe/ivy-releases/org.scala-tools.sbt/sbt-launch/0.11.0/sbt-launch.jar).
2. Put the launcher jar somewhere in your path, such as `~/bin` or `/usr/local/bin`
3. Use a text editor to make a script called `sbt` that executes the jar. While you can modify the command line arguments to suit your own needs (to increase the maximum memory available to the JVM, for example), your script should look something like this:

    ```script
    java -Dfile.encoding=UTF8 -Xmx1536M -Xss1M \
         -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=256m \
         -jar `dirname $0`/sbt-launch.jar "$@"
    ```

4. Save this script to the same directory as the launcher jar, and set its permissions to be executable:

    `$ chmod a+x sbt`

### Cygwin
FIXME

### Windows
FIXME

## Discussion
SBT downloads most of its own dependencies on an as-needed basis, so the startup jar is the only thing you'll need to fetch yourself. 

## Further Reading and Resources 
See the [[Setup|Getting Started Setup]] page of the wiki for more information.
