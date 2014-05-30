---
layout: post
title:  "Using a directive for HawtIO"
date:   2013-10-02
comments: true
categories: [Open Source, HawtIO]
tags: [JavaScript, AngularJS, Directive, OSGI]
---
As I have written in an earlier post my recent project work allowed me to spend some time contributing to [hawtio](http://hawt.io). Out of personal interest I have migrated some older code I had to analyze OSGI dependencies to a hawtio plugin. It took me a bit to figure out how to do that living up to the Angular idea of MVC separation. As it turned the contribution ended up in two plugins:

* One tapping into the backend data collecting the OSGi metadata and preparing the graph data
* One plugin providing an Angular directive to render the graph data using a [D3 force graph](http://bl.ocks.org/mbostock/4062045) layout.

This article will look at some details of the rendering directive as that might provide some help for others trying to write a directive. This article is not intended as a Angular or Typescript tutorial, please refer to the [Angular directive documentation](http://docs.angularjs.org/guide/directive) or the [Typescript](http://www.typescriptlang.org/) documentation for that.

The complete code and users guide for the directive is located within the [hawtio project on github](https://github.com/hawtio/hawtio/tree/master/hawtio-web/src/main/webapp/app/forcegraph). You may want to read that bit first before you read on...

## Using the directive

Angular directives are used to take an arbitrary data structure from the model (in Angular that is the _$scope_).  In our case we want to be able to stick a data structure that representing a graph with nodes and edges into the directive and expect the directive to take care of all the required rendering magic, in that particular case operating the D3 API.

Assuming that our scope has a member called _graph_, we want to use the following HTML code to display it:

{% highlight html %}
<div hawtio-force-graph
  graph="graph"
  link-distance="100"
  charge="-300"
  nodesize="10"
  style="min-height: 800px" />
{% endhighlight %}

_hawtio-force-graph_ is the name of the directive to render the graph. The example above uses the graph data within the current controller's scope and passes that into the _graph_ attribute of the directive. You see that the directive has other attributes that will be available within the directive code to drive the rendering.

## Registering the Directive

Each directive must be registered with the Angular framework, so that it can be found whenever HTML code like the above is encountered. Registering a directive is as simple as instantiating a class and stick it into the Angular framework:

{% highlight ts %}
module ForceGraph {
var pluginName = 'forceGraph';

  angular.module(pluginName, ['bootstrap', 'ngResource', 'hawtioCore']).

  directive('hawtioForceGraph', function () {
    return new ForceGraph.ForceGraphDirective();
  });

  hawtioPluginLoader.addModule(pluginName);
}
{% endhighlight %}

It is best practice that the code for one plugin is organized within a corresponding module, in this case the modules name is _ForceGraph_. The actual code simply uses the module function of angular to create the module and the directive within. The name of the directive, in our case _hawtioForceGraph_ is what defines the name of the attribute within the _div_ element. The is specified in camel case within the code, but normally is used with dashes within the HTML code. For the name a function is registered that returns an instance of  the class implementing the directive.

At last the module must be registered with the hawtio plugin loader.

## A look at the directive class

The class is defined in it's own [file](https://github.com/hawtio/hawtio/blob/master/hawtio-web/src/main/webapp/app/forcegraph/js/forceGraphDirective.ts). The file contains a lot of D3 specific code to render the graph data structure accordingly. Relevant for the directive plumbing are just a couple of attributes within the directive class. These attributes correspond to the attributes within a directive definition object as specified in the Angular documentation.

{% highlight ts %}
module ForceGraph {

  export class ForceGraphDirective {

    public restrict = 'A';
    public replace = true;
    public transclude = false;

    public scope = {
      graph : '=graph',
      nodesize : '@',
      linkDistance : '@',
      charge : '@'
    };

    public link = ($scope, $element, $attrs) => {

      $scope.trans = [0,0];
      $scope.scale = 1;

      $scope.$watch('graph', (oldVal, newVal) => {
        updateGraph();
      });

// ... Many lines ommitted

    };
  };
}
{% endhighlight %}

Let's walk through these bits one by one:

1. The _restrict_ attribute defines the context in which the directive can be used. "A" in this case allows the usage as an attribute, which is also the default for directives.
1. The setting _replace = true_ specifies that the _div_ element containing the directive is replaced with whatever the directive generates.
1. _transclude = false_ indicates that content within the div element will not be available within the directive. In our case that makes sense as the relevant graph data is passed as a scope attribute.
1. The _scope_ attribute defines an isolated scope. In other words, only stuff that we stick into the scope from within the directive or pass in as attributes will be available within the $scope object variable. Within the scope, _graph: "=graph"_ denotes that the graph object of the outer scope is passed into the _graph_ variable of the directive. All attributes simply referenced via an '@' on the right hand side will be passed in as values from div element attributes with the same name.
1. The link function actually performs the rendering. In our case we set up a watch on the graph variable of the directive, so that angular executes updateGraph() whenever that variable changes. As the variable is bound to the outer scope, the graph will be re-rendered whenever the underlying graph data changes.

## The end

That concludes our quick walk through the force graph directive plugin. Next time I will walk though the data factory that provides the graph data and how that factory is used to create the graph data structure.





