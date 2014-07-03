---
layout: page
title: Project Blended
headline: "Akka API example"
tags: [WoQ, Way of Quality, Blended, OSGi, Akka]
comments: true
---
## Actor based OSGi API

Assuming that we are building OSGi based Akka applications - or Akka based OSGi applications - with building blocks that provide subtrees of our Actor universe we also want a nice API that allows us to construct an actor bundle quickly. Here are some of the main tasks that are required for all actor bundles:

* Each actor bundle must expose atleast its main actor via the containers ActorSystem. To do so it must lookup the ActorSytem service, create the main actor and register it using the ActorSystem API.

* After the actor has been registered it must retrieve the configuration from the containers config directory and use it for completing it's own initialization.

## High level API walk through

Obviously, these are plumbing tasks that can be abstracted away into a nice API. Let's look at an example that is contained in the module [blended-mgmt-agent]({{ site.project.blended.github }}/tree/master/blended-mgmt-agent) and is based on the API defined in [blended-akka]({{ site.project.blended.github }}/tree/master/blended-akka).

Basically we are defining a bundle performs a periodic report of the most important container properties. For now it does so via the logging system, but in the near future it will report against a REST url (as soon as we got the spray integration working).

### A simple bundle actor

#### An actor in working state

The core task of reporting the container information every so often can be defined in a simple receive function:

{% highlight scala %}
  def working(implicit osgiContext: BundleContext) = LoggingReceive {

    case Tick => {
      log debug "Performing report"

      invokeService[ContainerIdentifierService, ContainerInfo]
        (classOf[ContainerIdentifierService]) { idSvc =>
        new ContainerInfo(idSvc.getUUID, idSvc.getProperties.toMap)
      } pipeTo(self)
    }

    case ServiceResult(Some(info))  => log info s"$info"
  }
{% endhighlight %}

The idea is to regularly receive __Tick__ messages that trigger the retrieval of the current container info and its reporting. Note that the function is wrapped with __LoggingReceive__, so that messages can be traced by [Akka's debugging and logging](http://doc.akka.io/docs/akka/snapshot/java/logging.html) facilities. The interesting part of the method is how we actually get the ContainerInfo to report. The __invokeService__ method is defined in a trait [__OSGIActor__](https://github.com/woq/de.woq.osgi.java/blob/master/blended-akka/src/main/scala/de/woq/blended/akka/OSGIActor.scala) and is defined as

{% highlight scala %}
def invokeService[I <: AnyRef, T <: AnyRef](iface: Class[I])(f: InvocationType[I,T])
  : Future[ServiceResult[Option[T]]]
{% endhighlight %}

Basically we are saying *Look up an instance of a service implementing I, then perform f: I-> T on that instance and eventually hand me back the result.* As the result of that invocation is a Future, we can simply pipe the result to ourselves and process it with the second case branch. The invocation uses the loaner pattern and frees the service reference once the call has finished.

The implementation of invokeService is completely actor based and non blocking. Even if the service reference is temporarily unavailable because the bundle offering the service is restarted, the service invocation would still succeed. (Technically it creates a service tracker behind the scenes and handles the invocation when the service instance has come back).

#### Initializing the bundle actor

Now let's turn towards initializing the bundle actor, which is again captured inside a receive function:

{% highlight scala %}
case object Tick

private [MgmtReporter] var ticker : Option[Cancellable] = None

def initializing = LoggingReceive {
  case InitializeBundle(bundleContext) => {
    log info "Initializing Management Reporter"
    ticker = Some(context.system.scheduler.schedule(100.milliseconds, 60.seconds, self, Tick))
    context.become(working(bundleContext))
  }
}
{% endhighlight %}

As you can see, eventually the Akka OSGi facade sends an __InitializeBundle__ message to the bundle actor which the bundle actor must process. In our case we are just setting up the ticker and switch to the __working__ state. If you were to read configuration from the bundle actors configuration file, you could do so by

{% highlight scala %}
def initializing = LoggingReceive {

  case InitializeBundle(_) => getActorConfig(bundleSymbolicName) pipeTo (self)

  case ConfigLocatorResponse(id, cfg) if id == bundleSymbolicName => {
    // Perform config here and switch to working state
  }
{% endhighlight %}

### Creating the bundle Actor

To leverage the API described above, the bundle actor must be self typed to ```OSGIActor with BundleName```. The __OSGIActor__ trait is the entry to the actor based API around the basic OSGi bundle cobtext API whereas the __BundleName__ simply mixes in a trait exposing the bundle's symbolic name.

With that in mind, the definition of our __MgmtReporter__ is

{% highlight scala %}
class MgmtReporter extends Actor with ActorLogging { this : OSGIActor with BundleName =>
  // rest of class body
}
{% endhighlight %}

As suggested in the Akka documentation, we wrap this up in a companion object:

{% highlight scala %}
object MgmtReporter {
  def apply()(implicit bundleContext: BundleContext) = new MgmtReporter with OSGIActor with MgmtReporterBundleName
}
{% endhighlight %}

Note that we generally pass the bundle context around implicitly to make the API a bit nicer.

### Finally, activating the Actor

Last not least, we need to create the bundle activator so that the OSGi framework can call it:

{% highlight scala %}
// Simply expose the bundle's symbolic name so that it can be mixed in where required
trait MgmtReporterBundleName extends BundleName {
  override def bundleSymbolicName = "de.woq.osgi.akka.mgmt.reporter"
}

// The Activator that is called from the OSGi framework whenever the bundle is started or stopped.
class ReporterActivator extends ActorSystemAware with BundleActivator with MgmtReporterBundleName {
  def prepareBundleActor(): Props = Props(MgmtReporter())
}
{% endhighlight %}

The interesting part her is the ReporterActivator that mixes in the __ActorSystemAware__ trait. This trait has all the plumbing to look up the ActorSystem service and create and register the bundle actor. The only method that needs to be implemented is __prepareBundleActor__ and that simply uses the apply method of our bundle actor discussed earlier. Note that the bundle context is defined in the __ActorSystemAware__ trait as an implicit value, so it does not occur in our activator.

Finally, the __ReporterActivator__ must be referenced in the bundle manifest:

{% highlight text %}
Bundle-Version:\
  ${project.version}

Bundle-SymbolicName:\
  ${bundle.symbolicName}; singleton:=true

Bundle-Activator:\
  de.woq.osgi.akka.mgmt.reporter.internal.ReporterActivator

Import-Package: *
{% endhighlight %}

## Final Remarks

Perhaps its good to mention that the entire Management Reporter is defined in terms of internal packages, in other words no package is exported to the outside world. All communication happens in terms of messages of some API package.

The service regularly uses the Container identifier service, but doesn't do so permanently. That frees up references and may avoid obscure effects when the bundle is reloaded or upgraded.