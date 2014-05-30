---
layout: post
title:  "Using a Camel servlet in an OSGI container" 
date:   2013-05-07 
comments: true
author: Andreas Gies
categories: [Open Source, Camel]
tags: [Servlet, Camel, Java, OSGI]
---
The other day I was working on a use case that involves running Camel inside an OSGi container and for configuration I am using Apache Aries 1.0.0. As i have an OSGi HTTP Service already running in my container and need to support HTTP posts, I figured using the servlet component would make sense.

This having said, trying to use the configuration found here doesn't work out of the box. The main reason seems to be that CamelServlet is a class rather than an interface, so it can't be registered as an OSGi service easily. Googling around indicated that using the attribute *ext:classes* on the blueprint reference should do the trick, but I got errors from Blueprint not being able to create the service proxy.

At this point I stepped back and tried to understand what should happen and came up with a solution / workaround that I thought was worth sharing:

1. I need to gain access to the CamelServlet by OSGi means.
1. I need to stick a HTTP Registry into that servlet.
1. I need to define an endpoint that wires the Registry and the Servlet

## Getting access to an OSGi Service of type CamelServlet 

First, I have defined an Interface CamelServletProvider that has merely one method:

{% highlight java %}
public interface CamelServletProvider

  public CamelServlet getCamelServlet();
}
{% endhighlight %}

with an associated implementation:

{% highlight java %}
public class CamelServletProviderImpl implements CamelServletProvider {

  private CamelServlet camelServlet = null;

  public void setCamelServlet(CamelServlet camelServlet) {
    this.camelServlet = camelServlet;
  }

  @Override
  public CamelServlet getCamelServlet() {
    return camelServlet;
  }
}
{% endhighlight %}

The idea for this class is, that it lives alongside the CamelServlet and a CamelServletProvider is registered as an OSGi Service like this:

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<blueprint
  xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  >

  <bean 
    id="camelServlet" 
    class="org.apache.camel.component.servlet.CamelHttpTransportServlet" />

  <bean 
    id="servletProvider" 
    class="de.woq.camel.sib.servlet.internal.CamelServletProviderImpl">
    <property name="camelServlet" ref="camelServlet" />
  </bean>

  <service ref="camelServlet" interface="javax.servlet.Servlet" >
    <service-properties>
      <entry key="alias" value="/camel/services" />
      <entry key="matchOnUriPrefix" value="true" />
      <entry key="servlet-name" value="CamelServlet"/>
      <entry key="osgi.web.contextpath" value="/camel/services" />
    </service-properties>
  </service>

  <service 
    ref="servletProvider"
    interface="de.woq.camel.sib.servlet.CamelServletProvider">

    <service-properties>
      <entry key="servlet-name" value="CamelServlet"/>
    </service-properties>
  </service>

</blueprint>
{% endhighlight %}

Now we have a bundle that exposes a Camel Servlet as HTTP transport which I can use in endpoints to come in another bundle. We also have an OSGi handle to grab the properly typed CamelServlet that we will use for registering the routes later on.

## Create a fine tuned HTTP Registry for OSGi   

To make this work in OSGi I need a HTTP Registry, that I can stick into Blueprint and that reacts to CamelServletProviders coming and going 

The DefaultHTTPRegistry in Camel reacts only to instances of CamelServlet, which we can't use in the OSGi Context. Instead we clone the DefaultHTTPRegistry and mkae it listen to instances of our CamelServletProvider interface:

{% highlight java %}
public class  CamelServletHTTPRegistry implements HttpRegistry {
  private static final transient Logger LOG = LoggerFactory.getLogger( CamelServletHTTPRegistry.class);

  private static HttpRegistry singleton;

  private final Set consumers;
  private final Set providers;

  public  CamelServletHTTPRegistry() {
    consumers = new HashSet();
    providers = new HashSet();
  }

  /**
   * Lookup or create a HttpRegistry
   */
  public static synchronized HttpRegistry getSingletonHttpRegistry() {
    if (singleton == null) {
      singleton = new  CamelServletHTTPRegistry();
    }
    return singleton;
  }

  @Override
  public void register(HttpConsumer consumer) {
    LOG.debug("Registering consumer for path {} providers present: {}",
     consumer.getPath(), providers.size());
    consumers.add(consumer);
    for (CamelServlet provider : providers) {
      provider.connect(consumer);
    }
  }

  @Override
  public void unregister(HttpConsumer consumer) {
    LOG.debug("Unregistering consumer for path {} ", consumer.getPath());
    consumers.remove(consumer);
    for (CamelServlet provider : providers) {
      provider.disconnect(consumer);
    }
  }

  @SuppressWarnings("rawtypes")
  public void register(CamelServletProvider provider, Map properties) {
    LOG.info("Registering provider through OSGi service listener {}", properties);
    try {
      CamelServlet camelServlet = provider.getCamelServlet();
      camelServlet.setServletName((String) properties.get("servlet-name"));
      register(camelServlet);
    } catch (ClassCastException cce) {
      LOG.info("Provider is not a Camel Servlet");
    }
  }

  public void unregister(CamelServletProvider provider, Map<String, Object> properties) {
    LOG.info("Deregistering provider through OSGi service listener {}", properties);
    try {
      CamelServlet camelServlet = provider.getCamelServlet();
      unregister((CamelServlet)provider);
    } catch (ClassCastException cce) {
      LOG.info("Provider is not a Camel Servlet");
    }
  }

  @Override
  public void register(CamelServlet provider) {
    LOG.debug("Registering CamelServlet with name {} consumers present: {}",
     provider.getServletName(), consumers.size());
    providers.add(provider);
    for (HttpConsumer consumer : consumers) {
      provider.connect(consumer);
    }
  }

  @Override
  public void unregister(CamelServlet provider) {
    providers.remove(provider);
  }

  public void setServlets(List servlets) {
    providers.clear();
    for (Servlet servlet : servlets) {
      if (servlet instanceof CamelServlet) {
        providers.add((CamelServlet) servlet);
      }
    }
  }
}
{% endhighlight %}

The trick here is in the register / unregister method reacting to the CamelServletProvider interface.
Wire the CamelServlet instances and Http Registry to create routes

{% highlight xml %}

<?xml version="1.0" encoding="UTF-8" standalone="no"?>

<blueprint
  xmlns="http://www.osgi.org/xmlns/blueprint/v1.0.0"
  xmlns:camel="http://camel.apache.org/schema/blueprint"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="
  http://camel.apache.org/schema/blueprint
  http://camel.apache.org/schema/blueprint/camel-blueprint-2.10.3.xsd
">

  <!-- Create an instance of the cloned and modified registry -->
  <bean id="httpRegistry" class="de.woq.camel.sib.http.CamelServletHTTPRegistry" />

  <!-- Get hold of the Servlet Provider by its interface and servlet name -->
  <reference
    interface="de.woq.camel.sib.servlet.CamelServletProvider"
    filter="(servlet-name=CamelServlet)"
    timeout="1000">
    <reference-listener 
      ref="httpRegistry" 
      bind-method="register"
      unbind-method="unregister" />
  </reference>

  <!-- Create the servlet component -->
  <bean 
    id="servlet"
    class="org.apache.camel.component.servlet.ServletComponent">
    <property name="httpRegistry" ref="httpRegistry" />
  </bean>

  <camel:camelContext>

    <!-- Just convenience to stick file into a directory and have them
         posted to a URL. -->
    <camel:route>
      <camel:from uri="file:///tmp/woq-in" />
      <!-- /camel/services was the root context of my transport service above -->
      <camel:to uri="http://localhost:8080/camel/services/test" />
    </camel:route>

    <!-- The actual route -->
    <camel:route>
      <!-- Three /// are important, otherwise the endpoint will be
           registered with path "/" -->
      <camel:from uri="servlet:///test" />
      <camel:to uri="file:///tmp/woq-out" />
    </camel:route>

  </camel:camelContext>

</blueprint>
{% endhighlight %}
