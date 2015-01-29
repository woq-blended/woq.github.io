---
layout: post
title:  "A web application with Play, ScalaJS, ReactJS - Part IV"
headline: "Getting started with the Web client"
date:   2015-01-23
comments: true
author: Andreas Gies
categories: [Scala, Play, ScalaJS, ReactJS]
tags: [Scala, Play, ScalaJS, ReactJS, Castillo, Heroku]
---

In the recent articles within this series we have created a small Play application that provides a REST interface to
manipulate a simple business object. We have also set up a toolchain consisting of services that are available online 
and therefor we have minimized the overhead of setting up any development infrastructure ourselves. 

In this article we will finally kick off our web development. The overall goal for this article is to get a ReactJS 
component integrated into the application which displays the list of available seminars within a browser, so that we can 
extend that component in the next part to support all the operations of the REST service. 

The source code behind this article is in the project's github repository within the branch 
[03_InitialWebClient](https://github.com/CastilloSanRafael/castillo/tree/03_InitialWebClient).

So far in this series:

  * [Introduction]({%post_url 2014-12-28-play-scalajs-reactjs-webapp%})
  * [Part I - Initial setup]({%post_url 2015-01-03-castillo-01-initial-setup%})
  * [Part II - Exposing Business Objects]({%post_url 2015-01-18-castillo-02-business-object%})
  * [Part III - Pimping up the toolchain]({%post_url 2015-01-19-castillo-03-pimping-up%})
  
## Quick review of what's required 

The main theme of this article is to get everything in place, so that we can start developing in ScalaJS for the frontend
and also use Bootstrap to style the content of the application. For bootstrap we want to use *LESS* to have the maximum
flexibility in our stylesheets. 

For the ScalaJS code we need to set another module and configure the ScalaJS compiler. The compiled result must end 
up within the javascript resources of the Play application. 

Once we have the toolchain out of the way we will gear towards our first ReactJS component talking to the server side 
REST interface.

## Configuring Bootstrap for the project
  
Let's get started by getting bootstrap into the sources of our play application. As we want to use *LESS* we need to 
download the source code of bootstrap from the [Bootstrap homepage](http://getbootstrap.com/getting-started/#download). 

Unzip the zip with the bootstrap sources and within the unzipped sources you will find a `less` subdirectory. Assuming 
`$PRJ_HOME` is the root directory of the project, we need to copy the `less` folder of bootstrap to 
`$PRJ_HOME/server/app/assets/stylesheets/less/bootstrap`.

Bootstrap also comes with a collection of [Glyphicons](http://glyphicons.com) in the `dist/fonts` directory of the downloaded
bootstrap archive. If you plan to use those copy the folder `dist/fonts` to `$PRJ_HOME/server/app/assets/stylesheets/fonts`.

Your own *LESS* files would end up in `$PRJ_HOME/server/app/assets/stylesheets/less`. For convenience we will refer to 
this directory as `$LESS_SRC`.  Following the suggestions from `Extending Bootstrap`, create a file `_main.less` in 
`$LESS_SRC` with the following content:

{%highlight text linenos%}
@import "bootstrap/bootstrap";   // Import the original bootstrap less files
@import "_variables";            // override any variables for bootstrap/variables
@import "custom-styles";         // any custom styles we want to apply
{%endhighlight%}

Just for fun and to see whether the stylesheets are pulled in correctly we define in `_variables.less` 

{%highlight text%}
@brand-primary:         #123f64;
@body-bg:               blue;
{%endhighlight%}

and finally in `custom-styles.less`

{%highlight text%}
@brand-primary:         #123f64;
@body-bg:               blue;
{%endhighlight%}

The next step is to make the play application aware of the *LESS* files so that they are added to the application's 
resources. Luckily Play supports the compilation of *LESS* file with the `sbt-less` plugin, so we add this to our list 
of plugins in `project/plugins.sbt`: 

{%highlight text%}
resolvers += "Typesafe repository" at "http://repo.typesafe.com/typesafe/releases/"

addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.3.7")

addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "0.8.0-RC2")

addSbtPlugin("com.typesafe.sbt" % "sbt-less" % "1.0.0")

addSbtPlugin("org.scoverage" % "sbt-scoverage" % "1.0.1")

addSbtPlugin("com.codacy" % "sbt-codacy-coverage" % "1.0.0")
{%endhighlight%}

By default the plugin looks for a file `main.less`within `$LESS_SRC`, so we need to override that setting within our 
build file:
 
{%highlight scala%}
/**
 * The settings for the server module
 */
lazy val serverSettings = commonSettings ++ Seq(
  name := s"$appName-server",
  libraryDependencies ++= Dependencies.serverDeps,
  includeFilter in (Assets, LessKeys.less) := "__main.less"
)
{%endhighlight%}

As bootstrap also has javascript components, we add bootstrap as a dependency to the server module using 
[webjars](http://www.webjars.org), so that the required javascript libraries can be pulled in to our html templates 
via the assets controller:

{%highlight scala%}
lazy val serverDeps = Seq(
  "com.newrelic.agent.java" % "newrelic-agent" % Versions.newRelic,
  "org.webjars" % "bootstrap" % Versions.bootstrap,
  "org.scalatestplus" %% "play" % Versions.scalaTestPlus % "test"
)
{%endhighlight%}

What remains is to change the html template so that it uses the defined styles:

{%highlight html%}
<!DOCTYPE html>

<html>
    <head>
      <title>Just Play Scala</title>
      <link rel="stylesheet" href="assets/stylesheets/less/__main.css" >
    </head>
    <body>
      <h1>Just Play Scala</h1>

      <button type="button" class="btn btn-default" aria-label="Left Align">
        <span class="glyphicon glyphicon-align-left" aria-hidden="true"></span>
      </button>

      <script src="assets/lib/bootstrap/js/bootstrap.min.js"></script>
      <script src="assets/lib/jquery/jquery.min.js"></script>
    </body>
</html>
{%endhighlight%}

As you can see, all the resources are now available via the assets controller, which is in general used to serve static 
content for the Play application. Now we have everything in place give it a go and start the server:

{%highlight text%}
$ sbt
[info] Loading project definition from /Users/andreas/projects/play/castillo/project
[warn] There may be incompatibilities among your library dependencies.
[warn] Here are some of the libraries that were evicted:
[warn] 	* com.typesafe.sbt:sbt-native-packager:0.7.4 -> 0.8.0-RC2
[warn] Run 'evicted' to see detailed eviction warnings
[info] Set current project to castillo (in build file:/Users/andreas/projects/play/castillo/)
> project server
[info] Set current project to castillo-server (in build file:/Users/andreas/projects/play/castillo/)
[castillo-server] $ run

--- (Running the application, auto-reloading is enabled) ---

[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Ctrl+D to stop and go back to the console...)
{%endhighlight%}

Now we can navigate to [http://localhost:9000](http://localhost:9000) and we should see the same content as before, but 
bootstrapped ;):

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/FirstBootstrap.png"></a>
	<figcaption>Just Play Scala with Style</figcaption>
</figure>

Yes, it's not nice, but it shows the point ...

## Hello ReactJS

Now that we have the *LESS* files and Bootstrap out of the way for now we can turn to getting ScalaJS and ReactJS support into the
project. I tend to try and set up the tooling first and see if everything works before I turn to the actual programming 
task. This having said, let's use one of the examples of the ReactJS tutorial and stick it on the entry page of our
application - just to make sure that we can compile everything and assemble the application. 

### A Hello React Component 

The React component I selected simply says hello to someone and counts the number of seconds elapsed since the page 
has been displayed. In React terms the component's state is the number of seconds elapsed, while the person we are 
saying hello to is not. The latter is simply an initialization parameter, but doesn't change after the component has 
been instantiated. 

In other words: _Only things that change over time make up the React components' state._

The state of the first component is built from the previous state by increasing the number of elapsed seconds by 1 and 
we make sure that this happens every second. Finally we set up the component in the main method and we are done. 

_**Note:** I am not going into the details of ReactJS or ScalaJS here as we are only interested in setting up the build 
for now._

{%highlight scala linenos%}
package de.woq.castillo.client

import japgolly.scalajs.react.{ComponentScopeM, ReactComponentB}
import japgolly.scalajs.react.vdom.ReactVDom.all._
import org.scalajs.dom
import org.scalajs.dom.window
import scala.scalajs.js
import scala.scalajs.js.JSApp

object HelloReact extends JSApp{

  case class State(secondsElapsed : Long)

  class Backend {
    var interval : js.UndefOr[Int] = js.undefined
    def tick(scope : ComponentScopeM[_, State, _]) : js.Function =
      () => scope.modState(s => State(s.secondsElapsed + 1))
  }

  val helloCastillo = ReactComponentB[String]("appMenu")
    .initialState(State(0))
    .backend(_ => new Backend)
    .render((param,s,_) => div(
    h2(s"Hello from React, $param !"),
    h3(s"Time elapsed : ${s.secondsElapsed}s")
  ))
    .componentDidMount(scope =>
    scope.backend.interval = window.setInterval(scope.backend.tick(scope), 1000))
    .componentWillUnmount(_.backend.interval foreach window.clearInterval)
    .build

  def main(): Unit = helloCastillo("Andreas") render dom.document.getElementById("content")

}
{%endhighlight%}

The code for the component goes into a new module of our application `client` into the `src/main/scala` folder as it would
for any other Scala module. Within the client module again we create a symbolic link `shared -> ../shared`, so that 
the client also takes the shared Scala sources into account.

### Using the component with a Play view

Assuming the component above would be compiled into JavaScript and would be in the right place to be accessible from our 
entry page, we could just write the entry page like this:

{%highlight html linenos%}
<!DOCTYPE html>

<html>
  <head>
    <title>Just Play Scala</title>
    <link rel="stylesheet" href="assets/stylesheets/less/__main.css" >
  </head>
  <body>
    <div id="content"></div>

    <script src="assets/lib/jquery/jquery.min.js"></script>
    <script src="assets/lib/bootstrap/js/bootstrap.min.js"></script>
    <script src="assets/lib/react/react.min.js"></script>
    <script src="assets/javascripts/castillo-client-fastopt.js"></script>

    <script type="text/javascript">
      de.woq.castillo.client.HelloReact().main()
    </script>
  </body>
</html>
{%endhighlight%}

Apart from the scripts we have imported for bootstrap support we add the react javascript library to the mix. Finally,
our own client library has the ReactJS component compiled into javascript and we can call the main method of the component 
the create an instance of it and bind it to the `div` container named `content`.

### Adjust the build definition

In order to make that work, some adjustments to `project/CastilloBuild.scala` are required, so let's look at them one
by one:

#### Add the ScalaJS plugin to the project

To use ScalaJS we need to use the sbt plugin that allows to compile Scala to javascript, so we add this to `project/plugins.sbt`:

{%highlight scala%}
addSbtPlugin("org.scala-lang.modules.scalajs" % "scalajs-sbt-plugin" % "0.5.6")
{%endhighlight%}

#### Define the client module

The client module should look fairly familiar. We simply define a new project that lives in the `client` directory and
pull in the `clientSettings` that we defined in it's own variable for readability reasons. Also we need to pull in the 
`scalaJSSettings` that are pre-defined within the ScalaJS plugin. By using those settings the compiler for that module 
is set to the ScalaJS compiler, so that the compiled artifacts will be javascript.

{%highlight scala%}
/**
 * Define the client project as a ScalaJS application.
 */
lazy val client =
  project.in(file("client"))
  .settings(clientSettings:_*)
  .settings(scalaJSSettings:_*)

...

/**
 * The settings for the client module (ScalaJS)
 */
lazy val clientSettings = commonSettings ++ Seq(
  name := s"$appName-client",
  libraryDependencies ++= Dependencies.clientDeps.value
) ++ sharedDirSettings
{%endhighlight%}

We also need to make a small adjustment to the definition of the server module as it now need to aggregate the client 
module:

{%highlight scala%}
/**
 * Define the server project as a Play application.
 */
lazy val server =
  project.in(file("server"))
  .settings(serverSettings:_*)
  .enablePlugins(PlayScala) aggregate client
{%endhighlight%}

#### Define the client dependencies

Now let's have a look at the client dependencies. The client dependencies are actually dependencies to Scala libraries 
that have been compiled for javascript already. When we compile our own Scala code to javascript, the result will
contain the javascript code of these dependencies. In other words, these dependencies have nothing to do with the javascript 
libraries that we need to pull into our HTML. 

Let's use the HelloReact example to digest that a bit more. The client dependencies of the client are defined as 

{%highlight scala%}
lazy val clientDeps = Def.setting(Seq(
  "com.github.japgolly.scalajs-react" %%% "core" % "0.6.1",
  "com.github.japgolly.scalajs-react" %%% "test" % "0.6.1" % "test",
  "com.github.japgolly.scalajs-react" %%% "ext-scalaz71" % "0.6.1",
  "org.scala-lang.modules.scalajs" %%% "scalajs-dom" % Versions.scalajsDom,
  "com.scalatags" %%% "scalatags" % "0.4.0",
  "org.scala-lang.modules.scalajs" %%% "scalajs-jquery" % "0.6",
  "org.scala-lang.modules.scalajs" %% "scalajs-jasmine-test-framework" % scalaJSVersion % "test"
))
{%endhighlight%}

Consider the dependency to `scalajs-react/core`. This brings the Scala wrapper for ReactJS into the project and by using 
that wrapper the compiled of our project will also include the javascript code for the wrapper. No special import is 
required within the HTML page. However, the wrapper of course uses the ReactJS javascript library under the covers 
and this is not pulled into the compiled javascript of the project. In order to use the code that relies on the wrapper,
we have added `react.min.js` to the imported javascripts of our HTML page.

To have ReactJS available in our assets, we have added the webjars dependency to the server dependencies:

{%highlight scala%}
lazy val serverDeps = Def.setting(Seq(
  "com.newrelic.agent.java" % "newrelic-agent" % Versions.newRelic,
  "org.webjars" % "bootstrap" % Versions.bootstrap,
  "org.webjars" % "react" % Versions.react,
  "org.scalatestplus" %% "play" % Versions.scalaTestPlus % "test"
))
{%endhighlight%}

#### Adjust the server project 

With the changes so far we have defined a `client` module where the Scala source is compiled into javascript. Now we need to 
adjust the server module to achieve:

 * Whenever the server module is compiled we also want to compile the Scala code in the client module, so that the 
project's javascript library is updated. 
 * The javascript library must be present in the server's `public/javascripts` directory, so we will change the target 
directory for the compiler to achieve this.
 * We want to provide source maps for easier debugging, so we copy those as well to the server's `public/javascripts` 
directory. 

This gives us the following updated definition of the server project: 

{%highlight scala%}
lazy val serverSettings = commonSettings ++ Seq(
 name := s"$appName-server",
 scalajsOutputDir := (classDirectory in Compile).value / "public" / "javascripts",
 compile in Compile <<= (compile in Compile) dependsOn (fastOptJS in (client, Compile)) dependsOn copySourceMapsTask,
 dist <<= dist dependsOn (fullOptJS in (client, Compile)),
 stage <<= stage dependsOn (fullOptJS in (client, Compile)),
 libraryDependencies ++= Dependencies.serverDeps.value,
 includeFilter in (Assets, LessKeys.less) := "__main.less"
) ++ (
 // ask scalajs project to put its outputs in scalajsOutputDir
 Seq(packageExternalDepsJS, packageInternalDepsJS, packageExportedProductsJS, packageLauncher, fastOptJS, fullOptJS) map { packageJSKey =>
   crossTarget in (client, Compile, packageJSKey) := scalajsOutputDir.value
 }
) ++ sharedDirSettings
{%endhighlight%}

The only additional thing here is, that we use different optimizations for the final javascript when we build the development 
version or the final version. The latter uses Google closure compiler underneath to remove any unused javascript from 
the compiled library, so that it is smaller and can load faster. 

## A rudimentary component displaying the list of available seminars

So, we got a web page going with ScalaJS and React, now let's turn it into displaying a list of seminars.
This is quite simple going from the initial ReactJS example by defining the state of the component in terms
of our portfolio rather than elapsed seconds:

{%highlight scala%}
case class State(portfolio : List[Seminar])
{%endhighlight%}

This is possible because we have modified the build definition earlier to include the shared Scala sources 
within the Scala JS client. Now we can get rid of the dynamic portion of the backend code as we don't 
count the number of elapsed seconds anymore:

{%highlight scala%}
class Backend
{%endhighlight%}

Within the component we provide a very simple method to render the list of available seminars and 
we initialize the list of available seminars after the component has mounted:

{%highlight scala%}
val helloCastillo = ReactComponentB[String]("appMenu")
  .initialState(State(List()))
  .backend(_ => new Backend)
  .render((param,s,_) => {
    def createSeminar(seminar: Seminar) = div(
      h3(s"${seminar.details.title}"),
      p(s"Trainer ${seminar.details.trainer} -- ${seminar.details.duration} days")
    )

    div(
      h2(s"Our course portfolio"),
      div(s.portfolio map createSeminar)
    )
  })
  .componentDidMount(scope =>
    scope.setState(State(List(
      Seminar(1, SeminarDetails(
        title = "My cool Seminar",
        description = "React Seminar",
        trainer = "andreas@wayofquality.de",
        duration = 5
      ))
    )))
  )
  .build
{%endhighlight%}

After these changes we have a react component that will initialize itself with a static list of 
seminars upon initialization:

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/FirstPortfolio.png"></a>
	<figcaption>A preconfigured list of seminars</figcaption>
</figure>

## Communicating with the server

The final step for today's article is to get the list of available seminars from the server. This 
should be straight forward by replacing the code to initialize the list with a call to our REST 
interface and de-serialising the result into a list of seminars. 
 
Fortunately, Li Haoyi provides a ready-to-go library for simple JSON de(serialisation) within 
ScalaJS - [upickle](https://github.com/lihaoyi/upickle), so that the final component looks like 
this:

{%highlight scala%}
object HelloReact extends JSApp{

  case class State(portfolio : Seq[Seminar])

  class Backend

  val helloCastillo = ReactComponentB[Unit]("HelloCastillo")
    .initialState(State(List()))
    .backend(_ => new Backend)
    .render((_,s,_) => {
      def createSeminar(seminar: Seminar) = div(
        h3(s"${seminar.details.title}"),
        p(s"Trainer ${seminar.details.trainer} -- ${seminar.details.duration} days")
      )
    
      div(
        h2(s"Our course portfolio"),
        div(s.portfolio map createSeminar)
      )
    })
    .componentDidMount(scope => {
      val url = "/portfolio"
      Ajax.get(url).foreach { xhr =>
        println(xhr.responseText)
        val seminars = upickle.read[Seq[Seminar]](xhr.responseText)
        scope.setState(State(seminars))
      }
    })
    .buildU

  def main(): Unit = React.render(helloCastillo(), dom.document.getElementById("content"))

}
{%endhighlight%}

## Conclusion 

Now we have a Play application exposing a CRUD like REST service for seminars and a rudimentary React Component using 
that service to retrieve and display the list of available seminars. In the next article we will flesh out that 
component, so that it supports the other operations of the service as well. That will also allow us to dive a bit deeper 
into the ReactJS world.

We have also set up the first cut of the *LESS* compilation, so that we can work with our own copy of Bootstrap while 
we develop the application.

Further ahead we will then introduce Authentication and Authorization, so that access to data modification operations can 
be restricted. Once we reached that point we will turn to fleshing out the UI design of the application and see how 
ScalaJS can help us keeping things nice and tidy. 

As usual, feel free to contact me with questions, comments or corrections in case I missed something.
