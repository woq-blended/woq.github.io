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

## Hello ScalaJS

## Hello ReactJS

## Putting everything together 

## Conclusion 

