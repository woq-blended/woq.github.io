---
layout: page
title: Project Blended
headline: "Akka integration details"
tags: [WoQ, Way of Quality, Blended, OSGi]
comments: true
---

# General Overview

We are currently thinking that we will have one ActorSystem per container. From an OSGI perspective this ActorSystem lives in its own bundle and the bundle exposes the ActorSystem as an OSGI service, so that other bundles could obtain the service reference and do whatever they want to do in Akka world.

## How to configure Akka within OSGi ?

However, with respect to configuration, OSGI and Akka follow a different philosophy (also outlined in [this thread](https://groups.google.com/forum/#!topic/akka-user/gWXONNJ-cvw) on the Akka mailing list). The bottom line is that the normal Akka approach would be to load the Akka config tree when the Actor System is initialized. This implies that all bundles that may need to contribute to that config tree must be known to the Actor System bundle. Also, a reload of one of the contributing bundles could potentially lead to a _stale_ configuration (if we wouldn't restart the ActorSystem) or it would require a restart of the entrire ActorSystem and therefore all actors if we wanted to refresh the configuration.

Rolands suggestion was to have more than one ActorSystem in the container, each of which would see the bundles it requires and could therefore initialize its config tree as needed. From our perspective that feels a bit un-OSGI. We kind of like how the [OSGI ConfigAdmin](http://blog.osgi.org/2010/06/how-to-use-config-admin.html) service works. Each bundle would have it's own namespace (technically the _pid_) and rely on properties set in that space. The ConfigAdmin Service provides an interface for maintaining those values and also for bundles to monitor changes and react accordingly. His main point was that modifying the underlying config __after__ the ActorSystem has been started might break stuff ... badly.

The bottom line is that bundles can be started / stopped / upgraded etc. without other bundles being affected by the configuration changes that may occur with the bundle maintenance.

On the other hand we really like the HOCON notation for the config, which is so much clearer than plain property files and so much easier to use in combination with Scala.

Our current line of thinking is:

* If each bundle obeys to the rule that it only provides values within it's own namespace (to be defined what that actually is, but most likely something like a pid)
* and we were able to track bundles that have a __resource.conf__ embedded
* then we should be able to maintain a modularized config tree that could be used to resolve all contributed config values.

Definitely this is an area to needs some thought, but the basics are already there.

## High Level integration architecture

The high 30.000 feet view on the AkkaIntegration is shown in the picture below.

<figure>
	<img src="{{ site.url }}/images/projects/blended/akka/AkkaSystem.png">
	<figcaption>Akka based OSGi facade</figcaption>
</figure>

As you can see, the ActorSystem is exposed as an OSGI Service. Within the ActorSystem bundle we create the OSGI Facade that encapsulates the OSGI API. Other bundles would typically

1. obtain the ActorSystem by looking up it's OSGI service reference
1. obtain the OSGI Facade using the well known actor path for the facade
1. use the Facade using the defined [protocol](https://github.com/woq/de.woq.osgi.java/blob/master/blended-akka/src/main/scala/de/woq/blended/akka/protocol/Protocol.scala)

_Note: The OSGI API grows on an as needed basis and wouldn't support all OSGI use cases as of now. If you are missing something - why not clone the repo and contribute - or just create an [issue](https://github.com/woq-blended/blended/issues?state=open) so that your requirement wouldn't be lost._

Under the covers the OSGIFacade creates two child actors: One is the intended supervisor for all actors wrapping OSGI service references (an outside bundle would not use the service reference directly, but would use the actor wrapper instead). Whenever the OSGI facade responds to a __GetService__ request, it will create an actor for the underlying service and maintain it as a child of the references supervising actor. Technically this is achieved by forwarding the message from the facade to the references actor.

In a similar way we treat the configuration requests. A [ConfigLocator](https://github.com/woq/de.woq.osgi.java/blob/master/blended-akka/src/main/scala/de/woq/blended/akka/internal/ConfigLocator.scala) actor resolves the configuration asked for and return a config object to the requestor.

_The ConfigLocator is very likely the place that can be extended to handle the config in a more clever way._

Finally we provide a convenient way to retrieve the bundle actor for an arbitrary bundle. As [noted before](AkkaIntegration), a normal _actor bundle_ would not expose services in the OSGI sense, it would rather expose __one__ bundle actor as a direct child of the user guardian. Most likely that bundle actor will set up a sub-tree within the actor hierarchy.

Have a look at the [facade implementation](https://github.com/woq/de.woq.osgi.java/blob/master/blended-akka/src/main/scala/de/woq/blended/akka/internal/OSGIFacade.scala) to get an impression of how it works.


# Working with OSGI Services

## Service invocation

## Tracking Services

<figure>
	<img src="{{ site.url }}/images/projects/blended/akka/ServiceTracker.png">
	<figcaption>Tracking OSGi Services</figcaption>
</figure>

# API Details

## ActorSystemAware

### OSGIActor

### OSGI Aware Event Listeners
