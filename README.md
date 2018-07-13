
## Getting Started

First, sign-up for a Jira account on https://issues.sonatype.org. Login and create an issue for a new project. Sonatype will quickly approve the request.

### SBT (1.1+)

#### Build Settings
Add these plugins to `plugins.sbt`
```
addSbtPlugin("org.xerial.sbt" % "sbt-sonatype" % "2.3")
addSbtPlugin("com.jsuereth" % "sbt-pgp" % "1.1.1")
```

Integrate `build.sbt` settings

```
organization := "com.{website}.{project}"
homepage := Some(url("https://github.com/{username}/{project}"))
scmInfo := Some(ScmInfo(url("https://github.com/{username}/{project}"), "https://github.com/{username}/{project}.git"))

developers := List(
  Developer("{username}", "{fullName}", "{email}", url("{website}"))
)

licenses += ("{licenseName}", url("{licenseUrl}"))
/// e.g. ("Apache-2.0", url("http://www.apache.org/licenses/LICENSE-2.0"))
publishMavenStyle := true

// Add sonatype repository settings
publishTo := Some(
  if (isSnapshot.value)
    Opts.resolver.sonatypeSnapshots
  else
    Opts.resolver.sonatypeStaging
)
```
Next, we will create a file to store Sonatype credentials. *Do not commit your Sonatype credentials to Git*. In `project/sonatype.sbt`, we can add

```
credentials += Credentials("Sonatype Nexus Repository Manager",
  "oss.sonatype.org",
  "{username}",
  "{password}")
```
and ignore with `echo "\\nproject/sonatype.sbt" >> .gitignore` from project root.

#### PGP

Once Sonatype approves the project (watch for the Jira e-mails), we can now deploy the project.

Artifacts first need to be signed with PGP. If you do not have a PGP key, you can generate one with the `sbt-pgp` plugin:

```
sbt pgp-cmd gen-key
```
and then distribute it
```
pgp-cmd send-key ${keyname} hkp://pool.sks-keyservers.net
```

#### Deploy

To deploy artifacts to the Sonatype staging repository, use

```
sbt publishSigned
```

And to create a release,

```
sbt sonatypeRelease
```

It can take some time for the artifacts to be synced to Maven Central.

Be sure to read the e-mail from Sonatype and follow all directions such as replying on the ticket after initial release.

## FAQ

### `sbt pgp-cmd gen-key` fails to run

Check `ls ~/.gnupg/` for an existing key set. Remove/back-up these at your own discretion and then run the command again. There are plugin settings for changing the key storage location at the [sbt-pgp repository](https://github.com/sbt/sbt-pgp).

## References

SBT directions were originally adapted from a [blog post by Leonard Ehrenfried](https://leonard.io/blog/2017/01/an-in-depth-guide-to-deploying-to-maven-central/) accessed on July 13, 2018.
