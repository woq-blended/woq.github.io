---
layout: post
title:  "A web application with Play, ScalaJS and ReactJS"
headline: "Development notes"
date:   2014-12-28
comments: true
author: Andreas Gies
categories: [Scala, Play, ScalaJS, ReactJS]
tags: [Scala, Play, ScalaJS, ReactJS, Castillo]
---
As some of you may have noticed, we are setting up a seminar center in southern Spain where we will also host technical 
trainings. Our own focus these days is on the Scala language and the evolving Scala ecosystem, so it seems quite natural 
that we give Scala a try for the web site of our seminar house. Having heard about ScalaJS at Scala Exchange in December,
I want to build the web application with Play on the Server side and ScalaJS / React on the client side. Today I'll start 
a series of Blog entries describing the development of the application hoping it might proof useful for others trying 
something similar.

Having been a backend developer for most of my career so far I have limited experience with developing Web applications 
both on the client and the server side. Having a certain time limit I spent some time looking for applications of the shelf 
that would allow me to plan seminars and at the same time create a seminar calendar and maintain ongoing bookings - all 
integrated with the top nodge social networks. In about 6 weeks I have found nothing that I really liked. Either it was 
a small solution just to be used in the back office or it was too expensive for a small business like ours. 

In the end I wondered what a technology stack would look like if I was to create the application myself from scratch and
came up with [Scala](http://scala-lang.org), [Play](http://www.playframework.org), [ScalaJS](http://www.scala-js.org), 
[ReactJS](https://github.com/japgolly/scalajs-react) and [MongoDB](http://www.mongodb.org/) for persistence. Having heard 
about hosting platforms for Play applications I decided I wanted my application on Heroku with an attached MongoDB 
instance and also logging and monitoring plugins.

_**Note**_ : I have chosen and these particular list of add-ons as they have free plans throughout the development
 cycle and promise to grow on demand as the application matures.

## References 

There is a ton of references and examples out there on building these kind of applications, but personally I found 
reading up on the entire range of topics proofed quite challenging. Therefore I have decided to capture my findings 
and the articles / books I have found useful in this blog series. 

These are the ones I have started with:

### Product and Tool documentation
* [Play](https://www.playframework.com/documentation/2.3.7)
* [SBT](http://www.scala-sbt.org/0.13.5/docs/)
* [Heroku](https://devcenter.heroku.com/)
* [Heroku Papertrail](https://addons.heroku.com/papertrail) for application logging
* [Heroku New Relic](https://addons.heroku.com/newrelic) for application monitoring
* [Heroku Mongolab](https://addons.heroku.com/mongolab) for MongoDB hosting on Heroku

### [Scala JS tutorial](http://www.scala-js.org/doc/tutorial.html)

This is a general introduction to ScalaJS and walks through the setup of simple, pure ScalaJS projects. 

### [Hands-On ScalaJS](http://lihaoyi.github.io/hands-on-scala-js/#HandsOn)

An online book on ScalaJS with more complex examples. All examples are included in the online book itself. Most interesting 
after going through the ScalaJS basics is the chapter on cross building Scala code for execution within a JS environment 
or a JVM. 

### [ScalaJS/ReactJS examples](https://github.com/japgolly/scalajs-react/tree/master/gh-pages)

This is the homepage for scala-js/react completely built with scala-js/react itself. Most if not all of the examples 
of the original [ReactJS tutorial](http://facebook.github.io/react/docs/tutorial.html) can be found here in ScalaJS.

### Play for Scala 

| **Author**       || Peter Hilton       |
|                  || Erik Bakker        |
|                  || Francisco Canedo   |
| **ISBN**         || 978-1-61729-079-4  |
| **Published By** || Manning, 2014 |

### Play Framework essentials 

| **Author**       || Julien Richard-Foy |
| **ISBN**         || 978-1-78398-240-0  |
| **Published By** || Packt Publishing, 2014 |

### Extending Bootstrap 

| **Author**       || Christoffer Niska |
| **ISBN**         || 978-1-78216-841-6 |
| **Published By** || Packt Publishing, 2014 |

## Milestones 

As stated above the application under development shall help me to manage seminars, bookings and my interactions 
with participants. In general it should make my life easier and let me concentrate on the fun parts like 
writing software. As some of the web technology is rather new for me I have decided to approach the application 
in explorative sprints. Each of the first couple of sprints shall familiarize myself with a yet unknown technology 
or component in the context of the overall project.  

From today's perspective the sprints should be :

### Initial setup 

In the first step I am going to create a minimalistic Play application and configure it for deployment on Heroku. 
Further I will go through the applications configuration with respect to monitoring and logging and attach the 
MongoDB instance to the application. The only change from a coding perspective will be that the project will already 
be tailored towards a multi-module SBT project.

### The first Business Domain Object

In this sprint I will create a Scala case class for one of our business domain objects and explore how it can be 
used on the client and the server side. On the server I will implement a simple REST interface to interact with 
the business domain object. 

### The first React Component

Building on the REST interface of the last sprint I will implement a minimalistic React component to interact 
with the server. This shall give me some insight how Scala code can be used within the client and the server 
modules.

### Integrating Authentication and Authorization 

The next step is to restrict access for some operations on the server to certain users. I will use this sprint 
to explore the authentication and authorization patterns for this kind of applications. I will also explore 
the integration of popular social networks as authentication sources.

### Implementing persistence

The last _explorative_ sprint is to implement the persistence layer on top of MongoDB. In this sprint I will 
incorporate the appropriate drivers into the application and give the business domain objects a real persistent 
home.

### Completing the UI Design
 
Once I have the technical details out of the way I can concentrate on the UI design and create a responsive layout based 
on [Bootstrap 3](http://getbootstrap.com/). This will also include the refactoring towards multi-language support as the 
final application must support at least English, German and Spanish.

### Finishing touches

The sprints so far should have covered the technical challenges around building a web application in pure Scala. 
Now the implementation of the remaining business domain objects and the associated UI components should be more or 
less straight forward.
