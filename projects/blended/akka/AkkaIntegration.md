---
layout: page
title: Project Blended
headline: "Akka - OSGi integration"
tags: [WoQ, Way of Quality, Blended, OSGi]
comments: true
---

We are currently working on a tighter integration of [Akka](http://akka.io) into our OSGi based container. The motivation behind is that within an OSGi container concurrency is a given and we found several times that code complexity increases for more sophisticated applications. We have used Scala before if only for smaller prototypes and utilities. The use of Akka is new for us, though with some background in functional programming and experience in message based solutions it seems a natural fit.

# General thoughts on Akka on OSGi

## One Actor System per container

At an early stage we have decided to have one ActorSystem in our container that is exposed as an OSGi service.

## Getting the most out of the frameworks

From our perspective the power of OSGi comes with the modular approach and - when done correctly with the separation between interfaces and implementation. In a normal OSGi development, a bundle would expose just the API for one or more services in terms of their interfaces, while the implementation is left to the service provider bundles. Of course there are different shades of separation, but in the pure form outlined here it allows swapping and/or upgrading implementations without the users of the API's being aware of the change. In fact we have started the explorative work to swap some of our Java based bundles with Scala/Akka bundles and are quite happy with the results so far.

If you consider bundles from an Akka perspective, a bundle typically would register one or more actors with the overall ActorSystem, so that other actors in potentially other bundles can interact with them. This having said, with Akka and OSGi combined, the interface definition becomes a set of immutable message definitions and the actors would use those messages as a communication protocol with the outside world.

Typically, actor bundles would not expose their actors in terms of OSGi services. In our work we found that navigation the actor tree is done best by using the Akka facilities. This can be easily achieved by using the exposed ActorSystem as an entry point into the tree and navigating it with proper search queries. However, if required you could always define search functions that look up your actors from the ActorSystem.

## Akka / OSGi conventions used in this project

In our initial work we decided to go with some conventions and believe they feel quite natural to OSGi-fied developers:

### Being strict on the bundle type

A bundle is __either__ an actor bundle or an OSGi bundle. In other words a bundle that exposes actors via the ActorSystem would not expose services via OSGi. The actors might consume OSGi services, but ideally they would just talk to other actors in terms of messages. Though it is not required in a technical sense, we find that this separation helps to decompose an application into bundles.

### Being restrictive in terms of actor exposure

An actor bundle exposes __one__ top level actor that is the root of the actor subtree that this bundle may build. The top level actor is registered as a direct child of the user guardian within the containers' actor system using the bundle symbolic name as actor name. For example, if the bundle has the symbolic name __foo__, the top level bundle actor is located at __/user/foo__. We realize, that this could be a constraint that is to restrictive, but this allows us to define a nice API to quickly setup and configure an actor bundle. Note that this restriction does not hinder developers to __watch__ actors living in other bundles.

### Configuration of actor bundles

An actor bundle can be configured using [typesafe's config library](https://github.com/typesafehub/config). As we are using [Karaf](http://karaf.apache.org) as our main container base we felt that it is best to fall in line with a config setup that Karaf users would feel at home with. Each Karaf container has an __etc__ directory that normally holds property files with a __cfg__  extension. The content of these files is available to the OSGi bundles via an instance of the [ConfigurationAdmin](http://www.osgi.org/javadoc/r4v42/org/osgi/service/cm/ConfigurationAdmin.html) service.

With this in mind, we place the configuration files of our actor bundles in the __etc__ directory with the name

{% highlight text %}
${bundleSymbolicName}.conf
{% endhighlight %}

and let the API figure out how the configuration is resolved.

**_Note:_** *Currently we can only fallback to the configuration provided by Akka itself, not to any resource.conf files that may have come with other bundles, such as spray. As a workaround we are currently including those additional conf's in the actor bundle config until we found a solution that feels OSGi-fied.*

# Example

Have a look at [a simple example](AkkaApiExample.html) for a high level walk through of the API defined up to now.

# References

* The [Akka homepage](http://akka.io) is just a must for the latest documentation.

* Derek Wyatt's fantastic book **_Akka concurrency_** explains Akka from ground up and is an invaluable resource for getting into Akka.

* The [OSGi dining hakkers](https://github.com/akka/akka/tree/master/akka-samples/akka-sample-osgi-dining-hakkers) are a nice example of kickstarting an Actor System within Karaf and get started with testing based on [Pax Exam](https://ops4j1.jira.com/wiki/display/PAXEXAM3/Pax+Exam).

# Notes

* Does it make sense to implement the OSGI Facade as Akka extension
* Does it make sense to look into an extender pattern for handling configurations