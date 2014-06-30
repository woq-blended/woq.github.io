---
layout: page
title: Project Blended
headline: "How to get the code and get started"
tags: [WoQ, Way of Quality, Blended, OSGi]
---
## Code repostory

Our code is hosted at [github](https://github.com/woq-blended/blended). Why not fork it and have a look at our modules for yourself ? - We love pull requests ;).

## CI Server

We use [Jenkins](http://jenkins-ci.org) to build the project regularly.

* The [build job](http://ci.wayofquality.de:8081/jenkins/job/blended-build/) regularly builds and tests all the modules with in project.
* The [assembly job](http://ci.wayofquality.de:8081/jenkins/job/blended-assembly/) runs after the module build to create the sample container distribution and executes the integration tests.
* The [deploy job](http://ci.wayofquality.de:8081/jenkins/job/blended-deploy/) runs regularly every two hours or after the assembly job to rebuild and redeploy the [sample container](http://backend.wayofquality.de:8181/hawtio) (login _blended/blended_).

## Maven repository

All modules can be found in our [Artifactory instance](http://ci.wayofquality.de:8085/artifactory).

## Building

Apologies to all the __sbt__-lovers and __maven__-haters out there. We are still using maven for our builds even if our bundles are moving more and more towards scala.

### Build profiles

* The __parent__ profile just builds the parents that contain the versions of most of the dependencies and the overall plugin configurartions. All other modules inherit their core maven settings from one of the parents.
* The __build__ profile simply lists all of the modules so that they can be built in one go.
* The __assembly__ profile builds all defined containers. Currently there is only one example container, located in the module __de.woq.osgi.java.karaf.central__.
* The __itest__ profile runs all integration tests defined on top of the container packages.

### Executing the build

You can build individual modules by navigating to the directory containing the module and execute a normal maven build, such as

{% highlight bash %}
mvn clean install
{% endhighlight %}

Normally, you would just build from the top level directory and use one or more profiles in the maven command. When building for the first time, you have to build with __only__ the parent profile. After building the parent for the first time you can use any combination of the profiles listed above.

{% highlight bash %}
mvn clean install -P parent
mvn clean install -P build,assembly,itest
{% endhighlight %}

## Starting the sample container

Navigate to __blended-karaf-central/target__. Depending on your environment either unpack the zip- or tar.gz-archive. Then navigate into the extracted directory and start your container with

{% highlight bash %}
./bin/karaf
{% endhighlight %}

or with

{% highlight bash %}
.\bin\karaf.bat
{% endhighlight %}

The container will be started and a karaf shell within the current shell will become active displaying something like:

{% highlight text %}
[andreas@woqlinux de.woq.osgi.java.karaf.central-1.0.7-SNAPSHOT]$ ./bin/karaf
 __      __                __    ___            _ _ _
 \ \    / /_ _ _  _   ___ / _|  / _ \ _  _ __ _| (_) |_ _  _
  \ \/\/ / _` | || | / _ \  _| | (_) | || / _` | | |  _| || |
   \_/\_/\__,_|\_, | \___/_|    \__\_\\_,_\__,_|_|_|\__|\_, |
               |__/                                     |__/

Integration appliance powered by
- Scala            - Version 2.10.4
- Apache Karaf     - Version 2.3.5
- Apache Active MQ - Version 5.9.1
- Apache Camel     - Version 2.13.0
- hawt.io          - Version 1.3.0
- Akka             - Version 2.3.2
- Spray            - Version 1.3.1
- Neo4J            - Version 2.0.3

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.

karaf@root>
{% endhighlight %}

Please refer to the [Karaf documentation](http://karaf.apache.org/manual/latest-2.2.x/users-guide/using-console.html) to learn how to use the console.

## Embedded Management Console

You can also log in to [http:://localhost:8181/hawtio](http:://localhost:8181/hawtio) (user/password = woq/woq). That will display the hawtio management console as shown below. As soon as you bring in more components to the container that support hawtio (e.g. ActiveMQ or Camel), the management tabs for those components will become available. The screenshot below shows the OSGi dependency graph of the current container.

<figure>
	<img src="{{ site.url }}/images/projects/blended/hawtio-sample.png">
	<figcaption>Embedded Management console</figcaption>
</figure>

## Spray ready

Go to [http://localhost:8181/hello](http://localhost:8181/hello) and see a _beautiful_ result from the embedded [Spray](http://spray.io) framework.

## Live container

There is a live container running on our CI server. Feel free to browse to [http://backend.wayofquality.de:8181/hawtio](http://backend.wayofquality.de:8181/hawtio) if you don't want to build and start the demo container yourself.

The spray servlet sits there and waits at [http://backend.wayofquality.de:8181/hello](http://backend.wayofquality.de:8181/hello).

__Note: The demo container is rebuilt every 2 hours and restarts. It is just there to see if it works and to show it in action.__

