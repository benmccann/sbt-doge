sbt-doge
========

sbt-doge is a sbt plugin to aggregate across `crossScalaVersions` for multi-project builds, which I call partial cross building.

![sbt-doge](sbt-doge.png?raw=true)

Current implementation of `+` cross building operator does not take in account for the `crossScalaVersions` of the sub projects. Until that's fixed, here's an alternative implementation of it.

setup
-----

This is an auto plugin, so you need sbt 0.13.5+. Put this in `project/doge.sbt`:

```scala
addSbtPlugin("com.eed3si9n" % "sbt-doge" % "0.1.5")
```

usage
-----

First, define a multi-project build with a root project aggregating some child projects:

```scala
def commonSettings: Seq[Def.Setting[_]] = Seq(
  organization := "com.example.doge",
  version := "0.1-SNAPSHOT"
)

lazy val rootProj = (project in file(".")).
  aggregate(libProj, fooPlugin).
  settings(commonSettings: _*)

lazy val libProj = (project in file("lib")).
  settings(commonSettings: _*).
  settings(
    name := "foo-lib",
    scalaVersion := "2.11.1",
    crossScalaVersions := Seq("2.11.1", "2.10.4")
  )

lazy val fooPlugin =(project in file("sbt-foo")).
  dependsOn(libProj).
  settings(commonSettings: _*).
  settings(
    name := "sbt-foo",
    sbtPlugin := true,
    scalaVersion := "2.10.4",
    crossScalaVersions := Seq("2.10.4")
  )
```

Next run this from the root project:

```scala
> ;so clean; such test; very publishLocal
```

sbt-doge will break the above into the following commands and executes them:

```scala
> wow 2.11.1
> libProj/clean
> wow 2.10.4
> libProj/clean
> fooPlugin/clean
> wow 2.10.4
> wow 2.11.1
> libProj/test
> wow 2.10.4
> libProj/test
> fooPlugin/test
> wow 2.10.4
> wow 2.11.1
> libProj/publishLocal
> wow 2.10.4
> libProj/publishLocal
> fooPlugin/publishLocal
> wow 2.10.4
```

It is looking into `aggregate` of the current project, and for each aggregated project, running a loop for each `crossScalaVersions` and executing the passed in command. The currently supported prefixes are: `much`, `so`, `such`, and `very`.

`wow` is a better implementation of `++` that only affects the aggregated projects.

## CrossPerProjectPlugin

`CrossPerProjectPlugin` overrides sbt's `+` and `++` commands and uses doge's implementation that aggregates command respecting `crossScalaVersions` at each subproject.

can now be written as

    > ;+ clean; + test; + publishLocal

## strict aggregation

sbt-doge adds strict aggregation command `plz`. `plz 2.11.5 compile` will aggregate only the subproject that contains `2.11.5` in `crossScalaVersions`. The alias for `plz` command is `+++` for `CrossPerProjectPlugin`.
