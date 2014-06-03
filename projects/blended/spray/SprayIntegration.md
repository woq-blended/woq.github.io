---
layout: page
title: Project Blended
headline: "Spray Integration"
tags: [WoQ, Way of Quality, Blended, OSGi, Spray]
comments: true
---
## Why Spray in the container ?

In our projects each container usually has 2 primary interfaces to the outside world - 3 if you count the command shell via ssh. In integration scenarios where reliable messaging is required, [JMS](http://en.wikipedia.org/wiki/Java_Message_Service) based on [Apache ActiveMQ](http://activemq.apache.org) is our choice.

In today's application landscape REST interfaces play a more and more dominent role. To meet those requirements we have been using JAXRS resources in the past on top of an OSGI __HttpService__ instance. With the transition to Akka we believe that we require a foundation for our REST services that ties in with the Akka design and implementation principles. From what we have seen spray can be deployed inside an OSGi container with a small amount of effort.

## Spray in our container

For now we have chosen to run spray on top of the HttpService instance of our container, so we are using [spray-servlet](http://spray.io/documentation/1.2.1/spray-servlet/) under the covers. Essentially a bundle needs to register a __Servlet30ConnectorServlet__ as an OSGi service. Then the [PAX extender](https://ops4j1.jira.com/wiki/display/paxweb/Pax+Web) will pick up that servlet and register it with the underlying __HttpService__.

Effectively there are two ways to register a servlet instance as a service:

1. Create a war file that has a proper OSGi bundle manifest and install that bundle in the container. The PAX war extender will kick in and do the registration.
1. Programmatically create the servlet instance and use the OSGi API to register that as a service. In that case the PAX whiteboard extender will kick in and do the registration.

## Leveraging the actor based OSGi API

Our spray integration builds on top of our [akka integration](../akka/AkkaIntegration.html). Spray requires an actor that implements the route. This actor will be exposed as a bundle actor so that we can use the same configuration mechanism as for all other actor bundles.

The bundle actor needs to be wired up with the spray servlet and we are good to go.

## The spray OSGi plumbing

First we need to decide whether we want to expose the servlet using an OSGi-fied war file or do it programmatically. We have chosen the war file in our first attempt which works fine as long as the spray jar files are inside the __WEB-INF/lib__ directory or the spray default config is exposed via the ActorSystem's configuration (potentially by applying some classloader tweaking as discussed [here](https://groups.google.com/forum/#!topic/spray-user/lmcdBsxNzdQ)).

When we tried to follow a more modular (and in our opinion more OSGi-like) approach we found that injecting the configuration into the servlet context is not trivial. The main problem here seems to be timing: In our approach, the bundle actor starts by requesting it's configuration form the container's config directory. When using the war file approach, a servlet context listener needs that configuration information and must inject the spray settings into the servlet context before the route is actually started.

The above works - most of the time - but starting the servlet, looking up the config and injecting it into the servlet context happens concurrently. In about 2 out of 10 container starts we saw that our route wouldn't start because of the requirements in the servlet context.

Therefore we have decided to retrace and choose the programmatic approach for the time being. We will revisit the war file approach later. A side effect of the programmatic approach seems to be that the __WebBoot__ class name that is checked in the settings companions of spray is irrelevant. In other words, you can have value like ```foo``` in your config to make the ```require``` happy but otherwise it does not matter.

### An OSGi variant of the Servlet30ConnectorServlet

We had a look at the __Servlet30ConnectorServlet__ implementation and luckily its fairly well encapsulated. It relies on the actor system, the route actor and the spray connector settings to be injected via the servlet context. Withing OSGi these items will come from different places:

* The __ActorSystem__ is exposed as an OSGi service from some bundle. We need aquire that service instance and use it.
* The route actor will be our OSGi bundle actor.
* The connector setting need to derived from the bundle specific configuration. **_Note: Currently that leads to duplicating the spray default config in all bundle configs that use spray, but we will revisit our way of configuring bundles later._**

We capture the thing we need to inject in a trait:

{% highlight scala %}
trait SprayOSGIBridge {
  def actorSystem : ActorSystem
  def connectorSettings : ConnectorSettings
  def routeActor : ActorRef
}
{% endhighlight %}

Now we can define a connector servlet geared towards our requirements with a modified initialization:

{% highlight scala %}
class SprayOSGIServlet extends Servlet30ConnectorServlet { this : SprayOSGIBridge =>

  override def init(): Unit = {
    system = actorSystem
    serviceActor = routeActor
    settings = connectorSettings
    require(system != null, "No ActorSystem configured")
    require(serviceActor != null, "No ServiceActor configured")
    require(settings != null, "No ConnectorSettings configured")
    require(RefUtils.isLocal(serviceActor), "The serviceActor must live in the same JVM as the Servlet30ConnectorServlet")
    timeoutHandler = if (settings.timeoutHandler.isEmpty) serviceActor else system.actorFor(settings.timeoutHandler)
    require(RefUtils.isLocal(timeoutHandler), "The timeoutHandler must live in the same JVM as the Servlet30ConnectorServlet")
    log = Logging(system, this.getClass)
    log.info("Initialized Servlet API 3.0 (OSGi) <=> Spray Connector")
  }
}
{% endhighlight %}

So later on we can create an instance of __SprayOSGIServlet__ and simply mix in the things to inject.

## Show me the code

Have a look at a [simple example](SprayIntegrationExample.html) if you want to see how it works.

## References

* The [spray](http://spray.io) home page.