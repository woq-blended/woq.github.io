---
layout: page
title: Project Blended
headline: "Spray Integration Example"
tags: [WoQ, Way of Quality, Blended, OSGi, Spray]
comments: true
---
# A simple spray example

This is an overview of our spray integration so far and the goal is to get a simple [hello world example]({{ site.project.blended.github }}/tree/master/samples/de.woq.osgi.spray.helloworld) working in an OSGi container.

The service is just plain vanilla spray __HttpService__:

{% highlight scala %}
trait HelloService extends HttpService {

  val helloRoute = path("hello") {
    get {
      respondWithMediaType(`text/html`) {
        complete {
          <html>
            <body>Say hello to <i>spray routing</i> within OSGi.</body>
          </html>
        }
      }
    }
  }
}
{% endhighlight %}

We have followed the recommendation given in the spray documentation and have defined it in a trait that we will mix into our actor later on. As we have [said](SprayIntegration.html) we will use the spray servlet implementation to expose our friendly service. We also want to leverage our OSGi API to minimize the plumbing effort.

# Wire the route to the servlet

With the [integration](SprayIntegration.html) we have so far need to create a bundle actor that in it's initialization state sets up the servlet and transitions to running the route afterwards:

{% highlight scala %}
  def initializing = LoggingReceive {

    case InitializeBundle(_) => getActorConfig(bundleSymbolicName) pipeTo (self)

    case ConfigLocatorResponse(id, cfg) if id == bundleSymbolicName => {

      implicit val servletSettings = ConnectorSettings(cfg).copy(rootPath = Path(s"/$contextPath"))
      implicit val routingSettings = RoutingSettings(cfg)
      implicit val routeLogger = LoggingContext.fromAdapter(log)
      implicit val exceptionHandler = ExceptionHandler.default
      implicit val rejectionHandler = RejectionHandler.Default

      val actorSys = context.system
      val routingActor = self

      val servlet = new SprayOSGIServlet with SprayOSGIBridge {
        override def routeActor = routingActor
        override def connectorSettings = servletSettings
        override def actorSystem = actorSys
      }

      bundleContext.createService(
        servlet, Map(
          "urlPatterns" -> "/",
          "Webapp-Context" -> contextPath,
          "Web-ContextPath" -> s"/$contextPath",
          "servlet-name" -> "hello"
        ))

      context.become(LoggingReceive { runRoute(helloRoute) })
    }
  }
{% endhighlight %}

We decided to leave some boilerplate code that could be abstracted away in this method for illustration purposes. Let's take it one by one:

When the bundle is started our [akka OSGi plumbing](../akka/AkkaIntegration.html) will create the bundle actor for us and eventually send it an __InitializeBundle__ message.

When the bundle actor receives that message, it will request the configuration object and eventually we will get a __ConfigLocatorResponse__ with the config we have asked for.

Upon that message we need to set up spray. Creating the service and exposing it to OSGi is pretty straight forward at this point. The interesting part is why we need all the implicits. The answer to that question lies with the signature of the runRoute function:

{% highlight scala %}
def runRoute(route: Route)(
  implicit eh: ExceptionHandler, rh: RejectionHandler, ac: ActorContext,
  rs: RoutingSettings, log: LoggingContext
): Actor.Receive
{% endhighlight %}

As you can see we need to satisfy a couple of implicits before actually running the route. Without having looked up the full details, the default implementation requires the spray config to available in the ActorSystem config tree. In out case we have a bundle specific config and therefore we redefine the implicits as shown above.

After having everything activating the route is as simple as

{% highlight scala %}
context.become(LoggingReceive { runRoute(helloRoute) })
{% endhighlight %}

**_Note: This implicitly wires the route to the OSGi HttpService instance in the container._**

# Create the bundle activator

Thanks to our [akka integration](../akka/AkkaIntegration.html) this is as simple as:

{% highlight scala %}
trait HelloBundleName extends BundleName {
  def bundleSymbolicName = "de.woq.osgi.spray.helloworld"
}

class HelloActivator extends ActorSystemAware with HelloBundleName {
  override def prepareBundleActor() = Props(HelloRoute("woq"))
}
{% endhighlight %}

## Setting up spray in a Karaf container

This bundle is part of the [sample container setup]({{ site.project.blended.github }}/tree/master/de.woq.osgi.java.karaf.central).