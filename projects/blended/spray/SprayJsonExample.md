---
layout: page
title: Project Blended
headline: "Spray Integration"
tags: [WoQ, Way of Quality, Blended, OSGi, Spray]
comments: true
---
## A simple REST service using JSON marshallers

Our applications typically consist of a number of collaborating containers. They are not in a cluster, they just talk to each other via REST and JMS messages. Whenever a container is started the first time ever, sometimes it needs some additional set up in the data center. For example dedicated queues that have to be created or monitoring must be set up for that container.

We have seen in our [akka example](AkkaApiExample) how to set up an actor that regular gathers some container information. Instead of just logging the information the actual implementation posts the Json marshalled information to a REST url. Below we have tried to capture the essentials of providing the service that understands those posts.

You can find the code for this example [here](https://github.com/woq/de.woq.osgi.java/tree/master/de.woq.osgi.akka.mgmt.rest). *Please note, that this is not yet production ready code as error handling and the like are missing.*

## The data flow

First a container posts the Jsonified container info consisting of the container unique ID and some additional properties:

```json
{"containerId":"uuid","properties":{"fooo":"bar"}}
```

The management collector accepts the POST and notfies a container registry actor. In this example the registry actor simply logs the information to the event stream. The real implementation will notify all listening actors with the container info update. These actors are application specific and typically live in application specific bundles.

Finally the service responds with an OK message that just contains the containers unique id.

TODO: Add picture

## What should the route do ?

One of the most fascinating features of spray is how easily the services can be tested. Note that the test below relies on [spray-json](https://github.com/spray/spray-json) to handle the (un)marshalling to and from Json for our API objects.

```scala
class ManagementCollectorSpec
  extends WordSpec
  with Matchers
  with MockitoSugar
  with ScalatestRouteTest
  with CollectorService
  with SprayJsonSupport
  with ContainerRegistryProvider {

  "The Management collector" should {

    "handle a posted container info" in {
      Post("/container", ContainerInfo("uuid", Map("foo" -> "bar"))) ~> collectorRoute ~> check {
        responseAs[ContainerRegistryResponseOK].id should be("uuid")
      }
    }
  }

  override def registry = {
    implicit val bc = mock[BundleContext]
    system.actorOf(Props(ContainerRegistryImpl()))
  }

  override implicit def actorRefFactory = system
}
```

## A note on the bundle decomposition

We have split this simple scenario into 3 different bundles:

### The API / worker bundle

This bundle contains the container registry

1. The resource bundle
1. The client bundle

The management reporter is the client side in this scenario and therefore li




