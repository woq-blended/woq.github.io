---
layout: page
title: Project Blended
description: "Making OSGi based development easy"
tags: [WoQ, Way of Quality, Blended, OSGi]
---
This project is a collection of modules that can be used in the context of developing applications targeting OSGI runtime environments.
All components have been used in EAI-like scenarios on top of an [Apache Karaf](http://karaf.apache.org) container in version 2.3.5.

We are maintaining this project on an as needed basis with the intention to share the *public* parts of our development with a greater
audience, so that others either could use our modules as implementation and/or configuration examples or even base their container development
on our project.

Try it out
----------

Refer to this [short guide](Building.html) to get started with the components as quickly as possible.

High Level Components
---------------------

On a high level we are building on top of the following components with some minor add-ons:

### [Apache Karaf (2.3.5)](http://karaf.apache.org)

Apache Karaf provides robust building blocks on top of an OSGI framework like [Apache Felix](http://felix.apache.org/) or [Equinox](http://www.eclipse.org/equinox/). Among other things it provides hot deployment, ease of configuration, a very good command shell either in process or via ssh and a very good packaging mechanism, aka Karaf features.

### [Apache Camel (2.13.0)](http://camel.apache.org)

Apache Camel provides mediation and routing facilities and also a rich set of enterprise integration patterns  in our container. It supports a wide range of transport protocols and allows routes to be set up using the Java-, XML- or Scala DSL. That makes it very easy - not to say mandatory - to integrate Apache Camel in all kind of integration projects.

*We are currently looking into intergating Akka into our container framework and the [Akka Camel Support](http://doc.akka.io/docs/akka/snapshot/scala/camel.html) is one of the features that certainly needs investigation.*

### [Apache ActiveMQ (5.9.1)](http://activemq.apache.org)

Whenever we require JMS based messaging in our container we use ActiveMQ.

### [HawtIO (1.3.0)](http://hawt.io)

*Don't cha wish your console was hawt like me? I'm hawt so you can stay cool* ... Stolen from hawt.io's web site it describes nicely what it is aiming at. In a nutshell it is an extensible web console that helps us keeping our container under control and understand the message flows. As part of our development we have [contributed](http://wayofquality.de/index.php/en/blog/categories/listings/hawtio) to hawt.io.

### [Akka (2.3.2)](http://akka.io) - experimental integration

For quite some time we have been thinking about using Akka and Scala on top of OSGI. We have finally kicked off a bit of explorative work to wrap the core OSGI API's into an actor based API. An impression of what we are trying to achieve can be found [here](https://github.com/woq/de.woq.osgi.java/blob/master/de.woq.osgi.akka.mgmt.reporter/src/main/scala/de/woq/osgi/akka/mgmt/reporter/internal/MgmtReporter.scala).

*Please note that the Akka support is purely experimental at this stage. Very likely it's going to change drastically in a short time frame. The next things to look at is better error handling, support of more OSGI related API's and define some reusable OSGI Actor patterns.*

### [Spray (1.3.1)](http://spray.io) - experimental integration

Alongside with the support for Akka we are looking at integrating Spray into our container for building RESTful web services. So far we have just defined the features to include the spray bundles in the container. The current line of thinking is to use the Spray servlet bridge on top of an OSGI HTTP service to come with a simple demo.

*This is even more experimental than the Akka intergation.*

Detailed component overview
---------------------------

Have a look [here](ComponentOverview.html) to get a summary of all the modules and their content.

