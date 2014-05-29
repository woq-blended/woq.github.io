---
layout: post
title:  "Using a datafactory within HawtIO"
date:   2013-10-08
comments: true
categories: [Open Source, HawtIO]
tags: [JavaScript, AngularJS, DataFactory, OSGI]
---
In a previous post I have blogged how a directive can be implemented within hawtio to allow the encapsulation of complex rendering tasks based on the OSGI dependency view I recently contributed to the project. Today I am going to go a bit deeper into how the data is provided for that view.  Normally I have a test container running at http://ci.wayofquality.de:8888/hawtio/osgi/dependenciies if you want to have a look at a live view.

The full source code for the OSGI plugin and the force graph directive is located within the [hawtio](https://github.com/hawtio/hawtio/tree/master/hawtio-web/src/main/webapp/app) project at GitHub.

To summarize, the data factory within Angular provides the backend data to the $scope object, the diectives provide the rendering while the controller brings these two sides together and additionally provides the UI interaction logic.

## Registering the data factory

A data factory is a normal angular module that needs to be registered with the angular runtime. This registration usually happens in the plugin definition object. As you can see, a factory function is registered with a name, in our case osgiDataFactory. The function instantiates a class that provides the backend access functions.

{% highlight ts %}
angular.module('osgi', ['bootstrap', 'ngResource', 'ngGrid', 'hawtioCore'])
  .factory('osgiDataService', function (workspace: Workspace, jolokia) {
    return new OsgiDataService(workspace, jolokia);
 });
{% endhighlight %}

## Injecting the data factory into a controller

After the data factory has been registered, it can be used within the controller by the normal Angular injection logic as done by the OSGi dependency controller:

{% highlight ts %}
module Osgi {
  export function ServiceDependencyController(
  $scope,
  $routeParams,
  workspace:Workspace,
  osgiDataService: OsgiDataService) {
    ...
  }
{% endhighlight %}

Within the controller the variable osgiDataService can now be used a normal instance of a class. This instance is used from within the controller to retrieve the backend data whenever required. The OSGi dependency viewer encapsulates the assembly of the graph data in the class OsgiGraphBuilder. This is just done for the sake of code readability and not mandatory in other use cases.

The rest of the controller simply connects the UI elements of the web page to the $scope, so that the filetring options can be modified. Note, that the graph is only updated when the Apply button is pressed in order to avoid unnecessary rendering loops.

## Inside the data factory

Within hawtio the data factory will typically use jolokia to get to some JMX data from the container. That pretty much comes down to navigating to the correct JMX object and executing one or more JMX calls. Within the OSGi plugin we some helper methods for navigating the JMX tree. For example, the mbean that represents the OSGi service state can be retrieved like this:

{% highlight ts %}
export function getSelectionServiceMBean(workspace:Workspace):string {
  if (workspace) {
    // lets navigate to the tree item based on paths
    var folder = workspace.tree.navigate("osgi.core", "serviceState");
    return Osgi.findFirstObjectName(folder);
  }
  return null;
}
{% endhighlight %}

Note, that we need to find the folder first and then get the first (and only) MBean in that folder as that has a generated name. Once we have an MBean, we can execute an operation such as listServices() on it.

{% highlight ts %}
public getServices() {
  var services = {};

  var response = this.jolokia.request({
      type: 'exec',
      mbean: getSelectionServiceMBean(this.workspace),
      operation: 'listServices()'
  }, onSuccess(null));

  var answer = response.value;

  angular.forEach(answer, function (value, key) {
    services[value.Identifier] = value;
  });

  return services;
}
{% endhighlight %}

The result of the call in this case is a JSON map object and contains the information for all registered OSGi services in this particualr case. An excerpt of the response for my test container is:

{% highlight json %}
{
  "1": {
    "UsingBundles": [
      0,
      1,
      55,
      61
    ],
    "objectClass": [
      "org.osgi.service.packageadmin.PackageAdmin"
    ],
    "Identifier": 1,
    "BundleIdentifier": 0,
    "Properties": {
      "service.id": {
        "Value": "1",
        "Key": "service.id",
        "Type": "Long"
      },
      "service.ranking": {
        "Value": "2147483647",
        "Key": "service.ranking",
        "Type": "Integer"
      },
      "objectClass": {
        "Value": "org.osgi.service.packageadmin.PackageAdmin",
        "Key": "objectClass",
        "Type": "Array of String"
      },
      "service.vendor": {
        "Value": "Eclipse.org - Equinox",
        "Key": "service.vendor",
        "Type": "String"
      },
      "service.pid": {
        "Value": "0.org.eclipse.osgi.framework.internal.core.PackageAdminImpl",
        "Key": "service.pid",
        "Type": "String"
      }
    }
  }, ...
}
{% endhighlight %}

Now it's up to the data factory to process the JSON and provide it to the interested controller. The data factory is also a good place to enrich and/or filter the data, so that the controller can assume it has a ready to use dataset. The filtering in the data factory is not to be mistaken for applying a filter based on some data entered in the web page. That kind of filtering normally happens within the controller before the data is passed into the directives.

Sometimes you need to understand what kind of JSON data comes out of the JMX call and of course you can use hawtio to discover that. Let's use the example of retrieving the service stae to see how hawtio can help us. First, go to the JMX tab and navigate to the MBean in the tree on the left hand side. Then within the operations sub panel, select the operation to execute.

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/jmx_request.png"></a>
	<figcaption>JMX Request</figcaption>
</figure>

Now, the execution will show the JSON as it comes back from the call:

<figure>
	<img src="{{ site.url }}/images/{{ page.date | date: "%Y-%m-%d" }}/jmx_response.png"></a>
	<figcaption>JMX Response</figcaption>
</figure>

This concludes the run through the data factory. I hope, using a real plugin as example provided more value than confusion.
