---
layout: page
title: Project Blended
headline: "Intrgration Testing"
tags: [WoQ, Way of Quality, Blended, OSGi, Testing]
comments: true
---

# General Flow for integration tests

1. Create Docker file(s) based on base image for the container(s) under test
1. Run docker build for each dockerfile
1. start the image(s)
1. execute the integration test suite
1. tear down the environment

# Standard tests

# Useful links

[Continuous Integration Using Docker, Maven and Jenkins](http://www.wouterdanes.net/2014/04/11/continuous-integration-using-docker-maven-and-jenkins.html)

https://github.com/docker-java/docker-java 

http://docs.docker.com

[docker cleanup](https://github.com/dotcloud/docker/issues/3258)

[scala docker](https://github.com/almoehi/reactive-docker)

[private registry](http://www.activestate.com/blog/2014/01/deploying-your-own-private-docker-registry)

[Maven docker plugin](http://www.alexecollins.com/content/docker-maven-plugin/)

# Useful commands 

docker images | grep none | awk '{print "docker rmi -f " $3;}' | /bin/sh

docker rm $(docker ps -a -q)



