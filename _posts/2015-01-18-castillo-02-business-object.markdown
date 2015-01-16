---
layout: post
title:  "A web application with Play, ScalaJS and ReactJS - Part II"
headline: "Creating Business objects"
date:   2015-01-18
comments: true
author: Andreas Gies
categories: [Scala, Play, ScalaJS, ReactJS]
tags: [Scala, Play, ScalaJS, ReactJS, Castillo, Heroku]
---
After a couple of travelling days today I have some time to continue the prototyping for our Play application using 
ScalaJS and ReactJS on the client side. The initial setup has been discussed in the 
[first article]({%post_url 2015-01-03-castillo-01-initial-setup%}) of this series of blog entries. Today I will dive
into creating business object case classes and use them on the server side to implement a REST interface exposing the 
services regarding those business objects.

The source code behind this article is in the project's github repository within the branch 
[02_BusinessObject](https://github.com/CastilloSanRafael/castillo/tree/02_BusinessObject).

## Considerations for creating business object case classes 

The case classes for business objects will be used in two different contexts: The server side and also on the client 
side. This is important to know because those case classes need to be compiled and packaged differently for each of 
the target runtimes. In other words, the source code for those classes can be shared, while the compiled artifacts 
are different. One thing that struck me only after reading into the topic for a while is that even though the ScalaJS
compiler generates class files as well, these class files cannot be run on a JVM. For more information on this refer 
to [this article](http://www.scala-js.org/doc/sbt/run.html) within the ScalaJS documentation.

One solution that looks fairly straight forward is to keep the shared sources in a separate sub-directory. We won't 
treat that sub-directory as a module, but we will create a symbolic link from the server application respective the 
ScalaJS client module to the shared sources. To make sbt aware of the sources we will nee to add the sources that 
are available via the symbolic link as unmanaged sources to the client and server project. 

## Create the folder for the shared source files 

Of course you can create the directory structure from within your IDE, for illustration purposes I show the Linux 
commands to create the directory structure for the model classes. Basically they go into `shared/main/scala` and
within there into the package `de.woq.castillo.model`. 

{%highlight text%}
$ mkdir -p shared/main/scala
$ cd shared/src/main/scala/
$ mkdir -p de/woq/castillo/model 
$ cd de/woq/castillo/model/
{%endhighlight%}

## A simple business object 

Now we create the a file `Seminar.scala` with a rough definition of a Seminar object. A seminar object contains the 
description for a seminar that can be scheduled within the seminar calender. Most likely the model will be refined 
at a later stage, for now we just need a couple of fields to play with in our first steps. 

{%highlight scala linenos%}
package de.woq.castillo.model

case class Seminar (
  id          : Long,            // A unique identifier
  details     : SeminarDetails   // The seminar details
)

case class SeminarDetails(
  title       : String,          // A seminar title
  description : String,          // The course content
  trainer     : String,          // Who is the trainer
  duration    : Int              // The course duration in days
)
{%endhighlight%}

## Link the shared source files to the server project 

Now we need to make sure that this file is added to the server projects sources, so that we can define the service 
classes using the business object within the server module.

First we create a symbolic link `shared` within the server module:

{%highlight text%}
$ cd server
$ ln -s ../shared 
$ ls -al 
{%endhighlight%}

_**Note**: Create the links using relative directories, so they stay intact when the repository is checked out into a
different base directory._

Next, we need to tell sbt that we have additional sources within the server module, so we make some adjustments to the 
file `project/CastilloBuild.scala`. First, we capture the directory settings for the shared sources in it's own 
settings sequence, so that they can be added easily to the modules that use them. 

{%highlight scala%}
/**
 * Here we collect the settings for the shared source directories.
 */
lazy val sharedDirSettings = Seq(
  unmanagedSourceDirectories in Compile += 
    baseDirectory.value / "shared" / "main" / "scala"
)
{%endhighlight%}

Then we change the definition for the `server` module by adding the shared directory settings:

{%highlight scala%}
/**
 * Define the server project as a Play application.
 */
lazy val server =
  project.in(file("server"))
  .settings(serverSettings:_*)
  .settings(sharedDirSettings:_*)
  .enablePlugins(PlayScala)
{%endhighlight%}

You can review the full modified build file on 
[github](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/project/CastilloBuild.scala).

## Create and test a portfolio service

Now we can use the model classes to define a service for the seminar portfolio, which is for now a fairly straight 
forward CRUD-like service. The service definition goes into `Portfolio.scala` in the `src/main/scala` folder 
within the server module and there into the package `de.woq.castillo.services`. 

{%highlight scala linenos%}
package de.woq.castillo.services

import de.woq.castillo.model.{SeminarDetails, Seminar}

trait Portfolio {

  def list() : Seq[Seminar]
  def create(details : SeminarDetails) : Option[Seminar]
  def update(seminar: Seminar) : Option[Seminar]
  def delete(id: Long) : Option[Seminar]
  def get(id: Long) : Option[Seminar]
}
{%endhighlight%}

The full file is available on 
[github](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/app/de/woq/castillo/services/Portfolio.scala).

For now we don't dive into the details of persisting the seminar information as we concentrate on bringing up 
a REST service for the portfolio as quickly as possible to get an initial client going. The final portfolio 
service will also implement the trait above, but in conjunction with a persistent store. For now I have defined 
a very simple in memory 
[implementation](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/app/de/woq/castillo/services/PortfolioImpl.scala) 
of the portfolio.

Even though we are using a fake backend for now we all add some tests for the portfolio implementation. We can define 
the tests in a way that they are still valid when we switch to an implementation that actually uses a real persistence 
store. We will use [ScalaTest](http://www.scalatest.org/) for our tests - mainly because we have already used it in 
other projects already and it worked well for us. 

### Add Scalatest to the project 

To use ScalaTest, we add it as a test dependency to the server module:

{%highlight scala%}
lazy val serverDeps = Seq(
  "com.newrelic.agent.java" % "newrelic-agent" % Versions.newRelic,
  "org.scalatest" %% "scalatest" % Versions.scalaTest % "test"
)
{%endhighlight%}

### A simple specification for the Portfolio

We create a simple 
[specification](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/test/de/woq/castillo/services/PortfolioSpec.scala)
for the basic portfolio operations. Within a play server module the specification files go into the subdirectory `test`
rather than `src/scala/test`. After adding the specification we can execute them with sbt:
 
{%highlight text%}
$ sbt clean test
...
[info] Done updating.
[info] Compiling 7 Scala sources and 1 Java source to /Users/andreas/projects/play/castillo/server/target/scala-2.11/classes...
[info] Compiling 1 Scala source to /Users/andreas/projects/play/castillo/server/target/scala-2.11/test-classes...
[info] PortfolioSpec:
[info] A portfolio service
[info] - should allow to look up a seminar by it's id
[info] - should allow to delete a seminar by it's id
[info] - should allow to update a seminar with it's id
[info] - should allow to retrieve the list of seminars
[info] ScalaTest
[info] Run completed in 700 milliseconds.
[info] Total number of tests run: 4
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 4, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[info] Passed: Total 4, Failed 0, Errors 0, Passed 4
[success] Total time: 8 s, completed 16-Jan-2015 22:29:52
{%endhighlight%}

## Define and test the REST interface
 
## Conclusion
