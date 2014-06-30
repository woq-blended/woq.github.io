---
layout: page
title: Project Blended
description: "The components in more detail"
tags: [WoQ, Way of Quality, Blended, OSGi]
---
## [blended-activemq-brokerstarter]({{ site.project.blended.github }}/tree/master/blended-activemq-brokerstarter)

When we use an ambedded ActiveMQ Broker in our container we like to deploy it by dropping a blueprint xml file into the containers deploy folder. However, this seems to cause problems in some restart scenarios when the broker still has durable subscriptions and subscribers try to reconnect while the broker is yet fully started. This manifests in duplicate subscription id's. The brokerstarter is a small wrapper around the configured ActiveMQ broker that registers a connectionfactory to that broker only after the broker has completed it's own strtup process.

As intracontainer subscribers rely on that connection factory either by an OSGI Service tracker or the blueprint injection mechanism, those subscribers will only reconnect once the connection factory is registered successfully.

## [blended-akka-itest]({{ site.project.blended.github }}/tree/master/blended-akka-itest)

This is an eaxample writing integration tests against one of our packaged containers using [Pax Exam 3.0](https://ops4j1.jira.com/wiki/display/PAXEXAM3/Pax+Exam).

## [blended-mgmt-agent]({{ site.project.blended.github }}/tree/master/blended-mgmt-agent)

The management reporter regular retrieves the container info and reports it to a given HTTP REST url. This is one of the first bundles we implement based on Scala and Akka.

## [blended-mgmt-rest]({{ site.project.blended.github }}/tree/master/blended-mgmt-rest)

This is the counterpart of the management reporter. It implemets the REST service that accepts the HTTP posts and is one of the first REST services to be built on top of Spray.

## [blended-modules]({{ site.project.blended.github }}/tree/master/blended-modules)

This is a derivation of the [scalamodules](https://github.com/w11k/scalamodules) project. Most of the OSGI API has moved to the actor based API, but we have kept the rich interfaces for the bundle context, the service refrences and the filter and upgraded those to work with Scala 2.10.

## [blended-persistence]({{ site.project.blended.github }}/tree/master/blended-persistence)

This is a generic persistence layer that sits within our OSGi container. The Persistence Manager itself handles the Akka protocol that other bundles can use to maintain their data. Behind the scenes the persistence manager uses a peristence backend and is therefore shielded from the actual persistence technology used. The current implementation uses [Neo4J](http://www.neo4j.org) as a persistence backend, but others could be implemented as well.

## [blended-akka]({{ site.project.blended.github }}/tree/master/blended-akka)

The bundle that provides an ActorSystem to the OSGI container. It also provides an actor based wrapper around the core OSGI API.

## [blended-archetypes-project]({{ site.project.blended.github }}/tree/master/blended-archetypes-project)

This archetypes helps to create a new project for implementing a container. It creates the project structure, the minimal amount of bundles and configuration to get started with a reasonable Karaf container.

## [blended-camel-utils]({{ site.project.blended.github }}/tree/master/blended-camel-utils)

Using the Camel Servlet component on top of an OSGI HTTP service has its problems as pointed out in our [blog](/open%20source/camel/using-camel-servlets-within-osgi/). This module wraps up the solution presented in the blog entry in a reusable OSGI bundle.

## [blended-container-context]({{ site.project.blended.github }}/tree/master/blended-container-context)

The ContainerContext presents an interface to some of the container configuration elements that are frequently used by other services.

## [blended-container-id]({{ site.project.blended.github }}/tree/master/blended-container-id)

All of our containers generate a UUID to identify themselves when they are first started. The UUID is written back into the configuration files, so that consecutive starts will yield in the same UUID. The Container ID Service allows arbitrary properties to be associated with the given UUID. These properties are normally defined by the operations teams and help them to query for the container UUID they are interested in.

## [blended-container-registry]({{ site.project.blended.github }}/tree/master/blended-container-registry)

The container registry will define a lightweight REST interface to collect runtime information and metrics from remote containers. The information will be persisted within the registry container and used within in a hawtio plugin for visualization.

**_This functionality is in planning, but does not exist yet._**

## [blended-installer]({{ site.project.blended.github }}/tree/master/blended-installer)

Apache Karaf already provides a [service wrapper](http://karaf.apache.org/manual/latest-2.2.x/users-guide/wrapper.html) that allows the Karaf user to register the container as an OS level service via the Karaf shell. However, this requires the container to be started before it can be registered, which in certain automated roll out scenarios cant't be achieved. Therefore we have tweaked the Service Wrapper code to provide an installer that allows the registration without requiring the container to run.

## [blended-itestsupport]({{ site.project.blended.github }}/tree/master/blended-itestsupport)

This is a collection of classes that help with integration tests run against an installed and started container. Sometimes we prefer this simplified integration test approach as it does not modify the container under test as Pax Exam sometimes does.

## [blended-jaxrs]({{ site.project.blended.github }}/tree/master/blended-jaxrs)

This is an implementation of an OSGI extender listening on classes that implement the marker interface *JAXRSExtender*. Such a class would be a normal REST resource with the according jaxrs annotations. The extender wraps the resource in a servlet and registers it with the container's HTTP service.

## [blended-karaf-branding]({{ site.project.blended.github }}/tree/master/blended-karaf-branding)

Most containers want a customized branding that shows up when the container ist started. Our standard packaging allows to specify a branding module and this is our default branding module.

## [blended-karaf-central]({{ site.project.blended.github }}/tree/master/blended-karaf-central)

This is an example container that has most of our modules deployed. The built artifact from this module is a tar-file or a zip file that can be unzipped in a target location. The unzipped directory can then be used for running the container. Our normal packaging mode is an offline packaging, so that no internet connectivity is required to start up the container.

## [blended-karaf-features]({{ site.project.blended.github }}/tree/master/blended-karaf-features)

Apache Karaf has a [provisioning strategy](http://karaf.apache.org/manual/latest-2.3.x/users-guide/provisioning.html) centered around features. Essentially a feature is composed of zero or more subfeatures and zero or more bundles. Within this module we keep our own feature definitions. These features can be used by the feature family of commands in the Karaf shell, the karaf configuration files and the offline packaging mechanism to include the required files in the container archive.

## [blended-karaf-parent]({{ site.project.blended.github }}/tree/master/blended-karaf-parent)

This module is the parent for all modules that are container assemblies. Within this project, the only defined container is **de.woq.osgi.java.karaf.central**.

## [blended-jmx]({{ site.project.blended.github }}/tree/master/blended-jmx)

Essentially this module simply registers the platform MBEan server as an OSGI service. Among other things this is required by [jolokia](http://www.jolokia.org/) which is used in the [hawtio](http://hawt.io/) management console.

## [blended-parent]({{ site.project.blended.github }}/blended-parent)

This module has the maven definitions that all other modules inherit.

## [blended-testsupport]({{ site.project.blended.github }}/tree/master/blended-testsupport)

This is a collection of some classes useful in testing. We frequently use messages captured using [HermesJMS](http://www.hermesjms.com/confluence/display/HJMS/Home) in our unit and integration tests. One of the helper classes allows to create Camel messages from saved hermes messages so that they can be injected in camel test routes.

## [blended-util]({{ site.project.blended.github }}/tree/master/blended-util)

Just a bundle to collect some helper classes used by other bundles.

## [blended-spray]({{ site.project.blended.github }}/tree/master/blended-spray)

This bundle is based on the Akka integration and provides the API to easily create a spray based REST resource in our container.

**_This is in very early stages. Comments are welcome, don't even think to use it somewhere in production !!!_**

