---
layout: post
title:  "A web application with Play, ScalaJS, ReactJS - Part II"
headline: "Creating and exposing Business objects"
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

To kickstart the Play application we are more or less following the overall patterns in the book _Play Framework essentials_
and already gear the build files towards adding a ScalaJS frontend later on. Sharing sources between Scala in the JVM 
and Scala in the frontend we follow [Hands-On ScalaJS](http://lihaoyi.github.io/hands-on-scala-js/#CrossPublishingLibraries).

The source code behind this article is in the project's github repository within the branch 
[02_BusinessObject](https://github.com/CastilloSanRafael/castillo/tree/02_BusinessObject).

So far in this series:

  * [Introduction]({%post_url 2014-12-28-play-scalajs-reactjs-webapp%})
  * [Initial setup]({%post_url 2015-01-03-castillo-01-initial-setup%})

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

We do not add ScalaTest as a dependency, but the super set for testing Play applications with ScalaTest, 
[ScalaTest Plus Play](http://scalatest.org/plus/play).

### Add Scalatest to the project 

To use ScalaTest, we add it as a test dependency to the server module:

{%highlight scala%}
lazy val serverDeps = Seq(
  "com.newrelic.agent.java" % "newrelic-agent" % Versions.newRelic,
  "org.scalatestplus" %% "play" % Versions.scalaTestPlus % "test"
)
{%endhighlight%}

### A simple specification for the Portfolio

We create a simple 
[specification](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/test/de/woq/castillo/services/PortfolioSpec.scala)
for the basic portfolio operations. Within the play server module the specification files go into the subdirectory `test`
rather than `src/scala/test`. After adding the specifications we can execute them with sbt:
 
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

## Interlude: A look at the project and package structure 

Before we go into the code itself, we'll sort it into a preliminary package structure to get this out of the way for 
now. Very likely the package structure will change a bit when more and more business objects and services are added 
to the application. 

Normally the source code for a play application is located within the subdirectory `app`, the test code in `test`. There 
is nor particular reason to change those settings. Within the `app` directory we find the standard directories `controllers`, 
`views` and `assets`. Within `controllers` we will keep the controller classes which glue the REST interface to the 
backend services. 

Within `views` we will have the views of the application. As we are heading towards a ScalaJS frontend ideally we 
will end up with only one view loading the ScalaJS application in this directory.

We have decided to keep the protocol files defining the JSON (de)serialization and the backend services within their 
respective packages, namely `de.woq.castillo.protocol` and `de.woq.castillo.services`.

## Define and test the REST interface

Now the next step is to expose the service implementation via REST using JSON to serialize the data. To do so we need 
a controller serving the REST requests and define the JSON serializers and deserializers for the business objects. 

### Serializing and deserializing business objects 

For (de)serializing business objects with JSON Play provides an API that can be used to define the `Reads` and `Writes` 
objects for the business objects. The `Writes` objects are straightforward and can be defined using the `Json.writes` 
macro. 

For the `Reads` it is a bit more involved as we are adding some data validation to the deserializer. For example, 
all `String` fields have a minimum length of 1. Also the seminar duration is at least 1. 

We have split the business object in two case classes as this allows us to reuse the `Reads` of the `SeminarDetails` 
within the `Seminar` itself.

Also we have added the `contentTypeOf`-methods to define the content type to be JSON when the body of an HTTP request 
is set to either `Seminar` or `SeminarDetails`.

{%highlight scala linenos%}
package de.woq.castillo.protocol

import de.woq.castillo.model.{Seminar, SeminarDetails}
import play.api.libs.json._
import Reads._
import play.api.libs.functional.syntax._

object PortfolioJSON {

  implicit val readsSeminarDetails : Reads[SeminarDetails] = (
    ( __ \ "title").read(minLength[String](1)) and
    ( __ \ "description").read(minLength[String](1)) and
    ( __ \ "trainer").read(minLength[String](1)) and
    ( __ \ "duration").read(min[Int](1))
  )(SeminarDetails)

  implicit val writesSeminarDetails = Json.writes[SeminarDetails]

  implicit val writesSeminar = Json.writes[Seminar]

  implicit val readsSeminar : Reads[Seminar] = (
    ( __ \ "id").read(min[Long](1)) and
    ( __  \ "details").read[SeminarDetails]
  )(Seminar)

  
  // define the Content-Type for Seminars and SeminarDetails as JSON
  
  implicit def contentTypeOf_Seminar(implicit codec: Codec) =
    ContentTypeOf[Seminar](Some(ContentTypes.JSON))

  implicit def contentTypeOf_SeminarDetails(implicit codec: Codec) =
    ContentTypeOf[SeminarDetails](Some(ContentTypes.JSON))

}
{%endhighlight%}
 
### Define the controller to serve the REST requests 

Now we can define the 
[controller](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/app/controllers/PortfolioController.scala)
to map the requests to the backend services. The individual routes are defined by 
actions that map a particular request to the appropriate HTTP response. For example, the list operation is most 
straight forward:

{%highlight scala%}
val list = Action { Ok(Json.toJson(portfolio.list())) }
{%endhighlight%}

Here we simply retrieve the list of available seminars and serialize them to JSON. The result is returned as an `Ok` 
response. The serialization works because we have imported the implicits from `PortfolioJSON`:

{%highlight scala%}
import de.woq.castillo.protocol.PortfolioJSON._
{%endhighlight%}

An operation that has to read the request body as JSON is a bit more involved as we need to handle the case that 
the JSON does not match the business object or violates one or more of the validation rules within the `Reads` 
definition. For example, for the `create` operation returns an `Ok` response only if the object has been created 
successfully. 

It returns an `InternalServerError` if the JSON was valid, but the object couldn't be created and a `BadRequest` 
if the JSON couldn't be deserialized into the business object.

{%highlight scala%}
val create = Action(parse.json) {
  implicit request => request.body.validate[SeminarDetails] match {
    case JsSuccess(createItem, _) =>
      portfolio.create(createItem) match {
        case Some(course) => Ok(Json.toJson(course))
        case None => InternalServerError
      }
    case JsError(errors) => BadRequest
  }
}
{%endhighlight%}

To make the routes available via HTTP we need to add the mappings to `conf/routes`:

{%highlight text%}
# The routes for the seminar portfolio
GET     /portfolio      controllers.PortfolioController.list
POST    /portfolio      controllers.PortfolioController.create
GET     /portfolio/:id  controllers.PortfolioController.details(id : Long)
PUT     /portfolio/:id  controllers.PortfolioController.update(id: Long)
DELETE  /portfolio/:id  controllers.PortfolioController.delete(id: Long)
{%endhighlight%}

The full source code of the controller is available on 
[github](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/app/controllers/PortfolioController.scala).

### Adding some fake data

To make it worthwhile using the service we will create some fake data using the applications `Global` object for now:

{%highlight scala linenos%}
object Global extends GlobalSettings {
  
  val portfolio : Portfolio = PortfolioImpl

  override def onStart(app: Application): Unit = {
    super.onStart(app)
    
    if (portfolio.list().isEmpty) {
      portfolio.create(SeminarDetails(
        title = "Creating Play applications with a ScalaJS frontend",
        description = "The title says it all",
        trainer = "andreas@wayofquality.de",
        duration = 5
      ))
    }
  }
}
{%endhighlight%}

### Intermediate push to Heroku 

Now and then we can merge the changes to the master branch and push them to heroku:
 
{%highlight text%}
git push heroku master
{%endhighlight%}

After the application as been restarted, we can open a browser with 

{%highlight text%}
heroku open
{%endhighlight%}

and add `/portfolio` to the browser's url to see the list of courses in our portfolio. 

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/Portfolio.png"></a>
	<figcaption>First view of the Portfolio in REST</figcaption>
</figure>

### Testing the HTTP routes

Now that we have everything in place, let's have a look at how the HTTP routes can be tested. There are quite a few 
tests in the 
[specification](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/test/de/woq/castillo/app/PortfolioRestSpec.scala).

Basically we create a specification that runs an embedded version of our Play application which is used across all the 
test within the Suite. This is achieved by mixing in `OneServerPerSuite`. The `WsScalaTestClient` is mixed in to make 
HTTP calls against the embedded server. 

The only thing to note is the implicit conversion from a `Writes[A]` into a `Writable[A]`. This implicit conversion 
allows us to use instances of our business objects as message bodies, for example

{%highlight scala%}
WS.url(s"$portfolioBase/${seminar.get.id}").put[Seminar]( /* Seminar definition here */ )
{%endhighlight%}

With that in mind the first cut of the Specification looks like this:

{%highlight scala linenos%}
package de.woq.castillo.app

import de.woq.castillo.model.{Seminar, SeminarDetails}
import de.woq.castillo.protocol.PortfolioJSON._
import org.scalatest.{Matchers, OptionValues, WordSpec}
import org.scalatestplus.play.{OneServerPerSuite, WsScalaTestClient}
import play.api.http.Writeable
import play.api.libs.json._
import play.api.libs.ws.WS
import play.mvc.Http.Status._

import scala.concurrent.Await
import scala.concurrent.duration._

class PortfolioRestSpec extends WordSpec
  with Matchers
  with OptionValues
  with WsScalaTestClient
  with OneServerPerSuite {
  
  private[this] lazy val portfolioBase = s"http://localhost:$port/portfolio"

  private[this] val details = SeminarDetails(
    title = "Yoga and IT",
    description = "Transcend to new programming skills",
    trainer = "andreas@wayofquality.de",
    duration = 5
  )

  // Transform a Writes[A] into a Writable[A]
  implicit private[this] def jsWriteable[A](
    implicit wa: Writes[A], wjs: Writeable[JsValue]
  ): Writeable[A] = wjs.map(a => Json.toJson(a))
  
  private[this] def executeWsRequest(request : WSRequestHolder) =
    Await.result(request.execute(), 2.second)
  
  private[this] def createSeminar(details : SeminarDetails) = {
    
    val result = executeWsRequest(
      WS.url(portfolioBase)
        .withBody[SeminarDetails](details)
        .withMethod("POST")
    )

    result.status should be (OK)
    validateJSON[Seminar](result.body)
  }
  
  private[this] def validateJSON[T](json : String)(implicit reads: Reads[T]) : Option[T] = {
    val result = Json.fromJson[T](Json.parse(json)).asOpt
    
    result match { 
      case None => fail("Failed to validate JSON response")
      case _ =>
    }
    
    result
  }
    
  ... All specifications here 
  
}
{%endhighlight%}

The full specification is available on 
[github](https://github.com/CastilloSanRafael/castillo/blob/02_BusinessObject/server/test/de/woq/castillo/app/PortfolioRestSpec.scala).

### Testing a simple get request

The simplest get request retrieves the list of all available seminars within the portfolio. The call has no parameter
and also no additional data within the request body. The `get` request actually returns a `Future`, so we simply
wait for the result and examine the HTTP status code. 

On `Ok` we examine the result whether it is a sequence of Seminars serialzed as JSON. 

{%highlight scala linenos%}
"provide the list of available seminars at /portfolio" in {
  val result = executeWsRequest(WS.url(portfolioBase).withMethod("GET"))
  result.status should be (OK)
  validateJSON[Seq[Seminar]](result.body)
}
{%endhighlight%}

### A test with a typed request body 

Updating a seminar with new details requires the new content to be passed as request body. With the definition we have 
in place this is quite simple. Note how the request body is set constructing the WS call by simply setting the business
object for the body. This works because we have braught the implicits defined in `PortfolioJSON` and also the implicit 
conversion from `Writes` into `Writables` in place. 

{%highlight scala linenos%}
"allow to update an existing seminar" in {
  createSeminar(details) match {
    case None => fail("Seminar for update not created")
    case Some(s) =>
      val update = executeWsRequest(
        WS.url(s"$portfolioBase/${s.id}")
          .withBody[Seminar](s.copy(details = details.copy(duration = 7)))
          .withMethod("PUT")
      )

      update.status should be (OK)
      val updated = validateJSON[Seminar](update.body)
      updated should not be None
      updated.foreach(_.details should be (details.copy(duration = 7)))
  }
}
{%endhighlight%}

Executing the specifocation with 

{%highlight text%}
sbt clean update test 
{%endhighlight%}

yields something like:

{%highlight text%}
$ sbt clean update test 

... Some omitted output

[info] PortfolioRestSpec:
[info] The portfolio REST service
[info] - should provide the list of available seminars at /portfolio
[info] - should allow to retrieve an existing seminar
[info] - should respond with a HTTP NotFound when trying to retrieve a non-existing seminar
[info] - should allow to create a seminar
[info] - should fail to create a seminar with wrong details
[info] - should allow to delete an object by it's id
[info] - should return HTTP 404 when trying to delete a non-existing seminar
[info] - should allow to update an existing seminar
[info] - should fail if updating the seminar with wrong details
[info] - should return HTTP 404 if trying to update a non-existing seminar
[info] PortfolioSpec:
[info] A portfolio service
[info] - should allow to look up a seminar by it's id
[info] - should allow to delete a seminar by it's id
[info] - should allow to update a seminar with it's id
[info] - should allow to retrieve the list of seminars
[info] ScalaTest
[info] Run completed in 2 seconds, 117 milliseconds.
[info] Total number of tests run: 14
[info] Suites: completed 2, aborted 0
[info] Tests: succeeded 14, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[info] Passed: Total 14, Failed 0, Errors 0, Passed 14
[success] Total time: 10 s, completed 17-Jan-2015 19:08:40
{%endhighlight%}
 
## Conclusion

In this part we have added the first business object to the application and have already prepared for reusing the case 
classes in the ScalaJS frontend. On top of the model we have provided a simple service exposing the operations on the
business object via REST. The (de)serializers between the business objects and JSON are fairly straight forward and 
can already be used for some basic data validation. 

We have also written some tests for testing the HTTP based routes and will add more as we go. In the next sprints we 
will first write the tests and then the actual code. In this first sprint it was a bit easier to explore the API's by 
writing the code first. 

The next part will finally turn to ScalaJS and ReactJS and define a first, but very simple UI for the portfolio 
service. 

As usual, I am happy to hear your thoughts and suggestions. 