---
layout: post
title:  "A web application with Play, ScalaJS, ReactJS - Part III"
headline: "Configuring for CI and more"
date:   2015-01-19
comments: true
author: Andreas Gies
categories: [Scala, Play, ScalaJS, ReactJS]
tags: [Scala, Play, ScalaJS, ReactJS, Castillo, Heroku, Travis-CI, Codacy, Waffle.IO]
---

After the finishing touches for the first rudimentary REST service and having it deployed on Heroku I thought now would 
be a good time to add some more sugar to the toolchain before actually diving into development with ScalaJS. Usually I 
have my own CI server to build my projects, but since I have Heroku as a PaaS it would be nice to use a CI as a service 
as well. Probably you have heard of [Travis-CI](http://travis-ci.org), a CI service that is free to use for public 
github repositories. 

While at it I also want to get some code quality analysis going and some test coverage metrics as well. Recently I have 
stumbled across [Codacy](http://www.codacy.com), an online service analyzing Scala code and also free for public github 
repositories. The latest version of Codacy has added support to display coverage information for the project as well.
 
Last not least, [Waffle.IO](http://waffle.io) provides a nice UI on top of the github issue tracker, so I will use that
as well. 

In this article I will walk through the setup of Travis-CI, Codacy and Waffle.IO. 

## Configuring Codacy

With [Codacy](http://codacy.com) getting started is very simple. You can sign in with your github account and simply 
select `Add Project` on the initial screen after login. You will pe presented with a choice to import a project from 
github, bitbucket or an arbitrary git url. 

After selecting github you will be presented with your github projects and you can select the one(s) that you would 
like to be checked. Please note that for now codacy only supports Scala, but that is just fine because we are developing 
a Scala project.
 
<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/Codacy-Add-Project.png"></a>
	<figcaption>Adding a github project for analysis in Codacy</figcaption>
</figure>

After adding the project to Codacy the initial analysis is started on the `master` branch with the default rule set. 
From within the project dashboard you will find a link `</> Code patterns` from where you can review and configure the 
ruleset that should be applied for future analyses. 

Within the project settings you can configure a post commit hook for github, so that after each commit a new analysis 
is triggered. 

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/Codacy-Add-Project.png"></a>
	<figcaption>Adding a github project for analysis in Codacy</figcaption>
</figure>

As usual one could argue about the applicability of the individual rules. Also there are some rules that could be satisfied 
simply by running [Scalariform](https://github.com/mdr/scalariform), but I haven't looked into setting this up for now.
Anyway, Codacy is simple enough to set up and get an impression how the code looks like and what could be improved, so 
why not have a go at it ?

_**Note:** After activating codacy I had 17 issues - all of unguarded usage of the `get` method of an `Option` type within my
test code. I have refactored the code [of the last article](https://github.com/CastilloSanRafael/castillo/tree/02_BusinessObject)
to address those issues and end up with a Codacy project. At a later stage I might be activating more rules._

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/Codacy-Results.png"></a>
	<figcaption>The Codacy Dashboard</figcaption>
</figure>


[Here](https://www.codacy.com/public/andreas_3098/castillo_2/dashboard) are the current Codacy results to look at.

## Configuring Travis-CI

Again, by signing into [Travis-CI](https://travis-ci.org) with your github account your public github repositories will 
become visible to Travis-CI and you can simply activate the Travis-CI build for any of your public projects by
flicking a switch on the repository overview. The default settings are in a way that a new build is triggered after
any code change on the master of the project.

Travis-CI tries to figure out the language and the build tool that should be used for the build from the underlying 
sources. However, it would try to use Scala and SBT if there is a `build.sbt` within the root folder of the project. 
Within our project we have moved to a Scala build file within the project folder, so the first build of the project 
failed as Travis-CI considered it to be a Ruby project. 

To configure the build, Travis-CI expects a file called `.travis.yml` in the root directory of the project. This file 
can be used to change the default settings and fine tune the build. We want to build a Scala project and refer to the 
[Travis-CI documentation](http://docs.travis-ci.com/user/languages/scala/) to understand what goes into the config file. 
 
The language must be set to `scala` and we also can select the Scala version and the underlying JVM version. If more
than one Scala version or JVM version is set in here, Travis-CI will create individual builds for each combination, 
which is quite handy to test against multiple runtime versions. 

The file that gets us going on Travis-CI is:

{%highlight yaml%}
language: scala
scala:
  - 2.11.4
jdk:
  - oraclejdk7
{%endhighlight%}

## Configuring for code coverage analysis

Next we want to have some basic coverage metrics and display them on the dashboard. This seems to be simple enough as 
Codacy is already prepared to accept uploads of cobertura results for the configured projects. All we have to do is to 
add the required plugins to our project and make sure the results are uploaded to Codacy. 

The [codacy documentation](https://www.codacy.com/docs?page=coverage) explains how the configuration works. 

Adding the additional plugins to `project/plugins.sbt` is simple enough: 

{%highlight text%}
resolvers += "Typesafe repository" at "http://repo.typesafe.com/typesafe/releases/"

addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.3.7")

addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "0.8.0-RC2")

addSbtPlugin("org.scoverage" % "sbt-scoverage" % "1.0.1")

addSbtPlugin("com.codacy" % "sbt-codacy-coverage" % "1.0.0")
{%endhighlight%}

Now there are two pieces missing: 

 * We need change the travis build so that it runs sbt with the tests and produces the coverage reports. 
 * We need to upload the reports to Codacy
 
For this we provide a `build.sh` in the root directory that calls sbt with the steps in the codacy documentation:

{%highlight bash%}
#!/bin/bash
set -ev
sbt clean coverage test
sbt coverageReport
sbt coverageAggregate
sbt codacyCoverage
{%endhighlight%}

To make Travis aware of the script and the `CODACY_PROJECT_TOKEN` we modify `.travis.yml` again (No, these are not my 
real tokens ;):

{%highlight text%}
language: scala
scala:
  - 2.11.4
jdk:
  - oraclejdk7
env:
  - APP_SECRET=foo CODACY_PROJECT_TOKEN=bar
script: ./build.sh
{%endhighlight%}

## Configuring Waffle.IO

Finally, we can pimp up the UI for the github issue tracker a bit using [Waffle.IO](http://waffle.io). Again, you see the 
list of your public github projects after signing in with your github account and you can select the project that you 
want Waffle.IO for. 

Waffle.IO creates a service within the target directory and asks whether you want to create a pull request for adding 
a Waffle badge to your `README.md`. 

Now, the project issues are displayed on a nice board and can be manipulated directly from that board:

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/Waffle.png"></a>
	<figcaption>The project issues served bei Waffle.IO</figcaption>
</figure>

## Make it visible 

The final step is to display everything we have configured on github itself. 

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/Badges.png"></a>
	<figcaption>All the badges on github</figcaption>
</figure>

This is very simple as Travis-CI, Codacy and Waffle.IO provide the markdown snippets within their project settings (if 
you have accepted Waffle.IO's pull request the Snippet is already in your `README.md`). Simply copy the snippets for the 
other badges to the top of your `README.md` and you are done.