---
layout: post
title:  "A web application with Play, ScalaJS and ReactJS"
headline: "Initial setup"
date:   2015-01-03
comments: true
author: Andreas Gies
categories: [Scala, Play, ScalaJS, ReactJS]
tags: [Scala, Play, ScalaJS, ReactJS, Castillo]
---
This is the first artcle in the blog post series developing a web application based on Scala, ScalaJS, React and Play 
and I will look at the initial set up. At the end of this first part I want to have a minimalistic Play application
that is configured as a multi-module SBT project (with only one module for now) which is deployable on Heroku and 
instrumented for basic monitoring. 

I assume that you are familiar with [git](http://git-scm.com) and [github](http://www.github.com) and already have 
set up git on your machine.

All code behind this part of the tutorial is available in the branch [01_Initial_Setup](https://github.com/CastilloSanRafael/castillo/tree/01_InitialSetup)
within the project's github repository.

## Scala, Play, Typesafe Activator, SBT and your favorite IDE

I will use the current versions of Scala (2.11.4), Play (2.3.7) and Sbt (0.13.7) for this project.

### Scala

For development you should have a JDK, Scala and SBT set up on your machine. A JDK is required as an execution 
environment for all the required tools. Though you could just downlowd the 
[Typesafe Activator](https://typesafe.com/community/core-tools/activator-and-sbt) to get started with developing 
applications in Scala and Play, personally I prefer a separate installation of the participating tools to have 
them at my fingertips when I need them.

You can download the desired Scala platform from [here](http://scala-lang.org/). Simply download the binaries for 
your operating system and unzip them into a directory of your choice. Make sure the **bin** directory of the unzipped 
binaries is on the search path for executables. 

On Linux or OS X i tend to create a symbolic link named **scala** to the unzipped binaries I want to use and put that 
on my path. For example within my OS X environment I have unzipped scala into **/opt** and therefore have 

{%highlight text%}
drwxr-xr-x@  8 root     wheel   272 18 Dec 12:27 .
drwxr-xr-x  34 root     wheel  1224 12 Dec 10:55 ..
drwxr-xr-x   8 root     wheel   272 11 Aug 23:55 X11
drwxr-xr-x   9 root     wheel   306 13 Oct 14:20 apache-maven-3.0.5
lrwxr-xr-x   1 root     wheel    18 13 Oct 14:20 maven -> apache-maven-3.0.5
lrwxr-xr-x   1 root     wheel    12 18 Dec 12:26 scala -> scala-2.11.4
drwxrwxr-x@  6 2000     2000    204 24 Oct 01:00 scala-2.11.4
drwxr-xr-x@  5 andreas  staff   170 13 Oct 19:18 zinc-0.3.5.3
{%endhighlight%}

After the successful installation and configuration you should be able to execute `scala -version from the command
and see something like this as a result:

{%highlight text%}
Andreass-MacBook-Air:~ andreas$ scala -version 
Scala code runner version 2.11.4 -- Copyright 2002-2013, LAMP/EPFL
Andreass-MacBook-Air:~ andreas$ 
{%endhighlight%}

### SBT

Next, you can download the desired sbt version from [here](http://www.scala-sbt.org/0.13/tutorial/Manual-Installation.html). 
I tend to use the manual installation and download the archive with the binaries for my platform and extract them into 
a directory of my choice. This will create a new directory named *sbt* which i normally change to *sbt-<version>* where 
version is the sbt version. This allows me again to create a symbolic link named *sbt* to the renamed directory and set 
up my path using the symbolic link. 

Now we would have something like 

{%highlight text%}
Andreass-MacBook-Air:~ andreas$ cd /opt/
Andreass-MacBook-Air:opt andreas$ ls -al
total 24
drwxr-xr-x@ 10 root     wheel   340  3 Jan 16:36 .
drwxr-xr-x  34 root     wheel  1224 12 Dec 10:55 ..
drwxr-xr-x   8 root     wheel   272 11 Aug 23:55 X11
drwxr-xr-x   9 root     wheel   306 13 Oct 14:20 apache-maven-3.0.5
lrwxr-xr-x   1 root     wheel    18 13 Oct 14:20 maven -> apache-maven-3.0.5
lrwxr-xr-x   1 root     wheel    10  3 Jan 16:36 sbt -> sbt-0.13.7
drwxr-xr-x@  5 andreas  staff   170  3 Jan 16:36 sbt-0.13.7
lrwxr-xr-x   1 root     wheel    12 18 Dec 12:26 scala -> scala-2.11.4
drwxrwxr-x@  6 2000     2000    204 24 Oct 01:00 scala-2.11.4
drwxr-xr-x@  5 andreas  staff   170 13 Oct 19:18 zinc-0.3.5.3
{%endhighlight%}

You can double check the sbt setup with `sbt --version` and should see something like 

{%highlight text%}
Andreass-MacBook-Air:opt andreas$ sbt --version
sbt launcher version 0.13.7
{%endhighlight%}

### Typesafe Activator 

Last not least you can download the [typesafe activator](https://typesafe.com/get-started). This is an application 
that gives you access to a ton of ready-to-go examples for the Typesafe platform. I will only use the activator 
to create a basic Play application later on. You can always create and explore any of the other examples to dig into 
a particular use case in more detail. Refer to the [activator documentation](https://typesafe.com/activator/docs) 
to learn how to browse the examples and use them as starting points for your new application.

If you have followed the instructions to get started with the activator, you should have the **activator** executable 
on your path and executing `activator ui` should start the activator backend and a browser with the the 
[activator ui](http://localhost:8888).

### Your favorite IDE 

Personally I use [IntelliJ IDEA](https://www.jetbrains.com/idea/) for developing in Scala. An alternative is the Eclipse 
based [Scala-IDE](http://scala-ide.org/). Install the IDE of your choice and we are ready to go.

## Create the initial Play application

Apparently the fastest way to get started with a new Play application is with one of the sample applications available 
within the activator. There are many sample applications touching quite a range of technologies. However, personally I 
felt that I would gain a deeper understanding if I started with the simplest template available and subsequently 
add the pieces I require for my application. 

The example I have chosen is [just-play-scala](https://github.com/julienrf/just-play-java#master). It is a minimal 
Play application with no dependencies. You can either instantiate the template from the activator ui within the browser
or use the activator command line. 

For now I'll stick with the command line version to create the application. Regardless which template you use you can 
use `activator new <app-name> <template-name` to instantiate the chosen template with an arbitrary name. In my case 
I have used 

{%highlight text%}
Andreass-MacBook-Air:play andreas$ activator new castillo just-play-scala
{%endhighlight%}

The result on the command line should look like this:

{%highlight text%}
Fetching the latest list of templates...

OK, application "castillo" is being created using the "just-play-scala" template.

To run "castillo" from the command line, "cd castillo" then:
/Users/andreas/projects/play/castillo/activator run

To run the test for "castillo" from the command line, "cd castillo" then:
/Users/andreas/projects/play/castillo/activator test

To run the Activator UI for "castillo" from the command line, "cd castillo" then:
/Users/andreas/projects/play/castillo/activator ui
{%endhighlight%}

The activator has instantiated the template in a subdirectory `<app-name>`. This is a ready to run sbt project, 
which you can open in your IDE. I won't go through the files that have been created, take your time to browse 
through them and understand what they are doing. The [initial post]({%post_url 2014-12-28-play-scalajs-reactjs-webapp%})
contains some references that can help digesting the generated files sufficiently.

### Running the initial application

Now you can navigate to the directory where the application has been generated and execute the application by

{%highlight text%}
sbt run
{%endhighlight%}

This will build the application and start the Play framework running it with a listener available on port 9000:

{%highlight text%}
Andreass-MacBook-Air:castillo andreas$ sbt run 
[info] Loading project definition from /Users/andreas/projects/play/castillo/project
[info] Updating {file:/Users/andreas/projects/play/castillo/project/}castillo-build...
[info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] Done updating.
[info] Set current project to castillo (in build file:/Users/andreas/projects/play/castillo/)
[info] Updating {file:/Users/andreas/projects/play/castillo/}root...
[info] Resolving org.fusesource.jansi#jansi;1.4 ...
[info] Done updating.

--- (Running the application from SBT, auto-reloading is enabled) ---

[info] play - Listening for HTTP on /0:0:0:0:0:0:0:0:9000

(Server started, use Ctrl+D to stop and go back to the console...)
{%endhighlight%}

You can now open a browser and navigate to http://localhost:900 and you should see a friendly **Just Play Scala** 
in the browser window as a result. I admit it is not much of an application, but it is nice to look at and nothing 
that keeps the brain busy for too long. Stop the server with `Ctrl-C` and we start cleaning up the initial set up.

### Initial git commit 

Examining the generated application you will find 3 files **activator, activator.bat, activator-launch-<version>.jar**. 
As you can run the project with sbt only I tend to get rid of these files and remove them before the initial commit. 
Also I find it is a good practice to run sbt clean and delete the logs before the initial commit. Also review the 
`LICENSE` file tha ´ has been generated. 

Now we are ready to create an initial git repository and perform the initial commit:

{%highlight text%}
Andreass-MacBook-Air:castillo andreas$ git init
Initialized empty Git repository in /Users/andreas/projects/play/castillo/.git/
Andreass-MacBook-Air:castillo andreas$ git add . 
Andreass-MacBook-Air:castillo andreas$ git commit -m "Initial commit." 
[master (root-commit) 7f42791] Initial commit.
 9 files changed, 128 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 LICENSE
 create mode 100644 app/controllers/Application.scala
 create mode 100644 app/views/main.scala.html
 create mode 100644 build.sbt
 create mode 100644 conf/application.conf
 create mode 100644 conf/routes
 create mode 100644 project/build.properties
 create mode 100644 project/plugins.sbt
Andreass-MacBook-Air:castillo andreas$ git status 
On branch master
nothing to commit, working directory clean
{%endhighlight%}

After the initial commit we create a repository on github and add it to our project as a remote repository. The code 
for the application lives at [https://github.com/CastilloSanRafael/castillo](https://github.com/CastilloSanRafael/castillo). 
To add the remote repository and push the initial commit: 

{%highlight text%}
Andreass-MacBook-Air:castillo andreas$ git remote add origin ssh://git@github.com/CastilloSanRafael/castillo.git
Andreass-MacBook-Air:castillo andreas$ git push origin master 
Counting objects: 16, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (14/14), done.
Writing objects: 100% (16/16), 2.71 KiB | 0 bytes/s, done.
Total 16 (delta 0), reused 0 (delta 0)
To ssh://git@github.com/CastilloSanRafael/castillo.git
 * [new branch]      master -> master
Andreass-MacBook-Air:castillo andreas$ git branch --set-upstream-to=origin/master
Branch master set up to track remote branch master from origin.
Andreass-MacBook-Air:castillo andreas$ git checkout -b 01_InitialSetup
Switched to a new branch '01_InitialSetup'
Andreass-MacBook-Air:castillo andreas$ git push origin --all
Total 0 (delta 0), reused 0 (delta 0)
To ssh://git@github.com/CastilloSanRafael/castillo.git
 * [new branch]      01_InitialSetup -> 01_InitialSetup
{%endhighlight%}

## Deploy the application to [Heroku](http://heroku.com)

Though the application we have is not much to look at I have decided to get the deployment and some basic application 
monitoring out of the way right from the beginning. There are some hosting alternatives for Play applications and I 
have decided to go with Heroku based articles and recommendations. For each application you deploy to Heroku you get 
750 instance hours. This basically that Heroku deployment is free as long as you use a single instance only, which is 
perfect for development. Later on the application can be scaled to multiple instances, which would be more than 750 
instance hours already with 2 instances. 

To get started with Heroku and Play you can follow the 
[Heroku Scala tutorial](https://devcenter.heroku.com/articles/getting-started-with-scala#introduction). Basically you 
have to 

* Install the heroku toolbelt, a simple command line utility to interact with Heroku 
* Add a Heroku repository as additional remote repository 
* Push the code to Heroku 
* Provide additional configuration if required 

After you have installed the heroku toolbelt for your platform, you can use `heroku login` to log into your heroku 
account; then you simply use `heroku create` to create the heroku git repository and add it as a remote repository to 
the project. This will add an additional remote repository to the project. Every time the master branch is pushed to 
that repository, the application will be rebuilt and started. 

{%highlight text%}
Andreass-MacBook-Air:castillo andreas$ heroku login
Your version of git is 1.9.3. Which has serious security vulnerabilities.
More information here: https://blog.heroku.com/archives/2014/12/23/update_your_git_clients_on_windows_and_os_x
Enter your Heroku credentials.
Email: andreas@wayofquality.de
Password (typing will be hidden): 
Authentication successful.
Andreass-MacBook-Air:castillo andreas$ heroku create 
Your version of git is 1.9.3. Which has serious security vulnerabilities.
More information here: https://blog.heroku.com/archives/2014/12/23/update_your_git_clients_on_windows_and_os_x
Creating stormy-crag-3594... done, stack is cedar-14
https://stormy-crag-3594.herokuapp.com/ | https://git.heroku.com/stormy-crag-3594.git
Git remote heroku added
Andreass-MacBook-Air:castillo andreas$ git push heroku master 
Counting objects: 16, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (14/14), done.
Writing objects: 100% (16/16), 2.71 KiB | 0 bytes/s, done.
Total 16 (delta 0), reused 0 (delta 0)
remote: Compressing source files... done.
remote: Building source:
remote: 
remote: -----> Play 2.x - Scala app detected
remote: -----> Installing OpenJDK 1.8... done
remote: -----> Downloading SBT... done
remote: -----> Priming Ivy cache (Scala-2.10, Play-2.3)... done
remote: -----> Running: sbt update
remote:        OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=512M; support was removed in 8.0
remote:        [info] Loading global plugins from /tmp/scala_buildpack_build_dir/.sbt_home/plugins
remote:        [info] Updating {file:/tmp/scala_buildpack_build_dir/.sbt_home/plugins/}global-plugins...
remote:        [info] Resolving org.scala-lang#scala-library;2.10.4 ...
remote:        [info] Resolving org.scala-sbt#sbt;0.13.5 ...

... Many more lines of build output 

remote: -----> Running: sbt compile stage
remote:        OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=512M; support was removed in 8.0
remote:        [info] Loading global plugins from /tmp/scala_buildpack_build_dir/.sbt_home/plugins
remote:        [info] Loading project definition from /tmp/scala_buildpack_build_dir/project
remote:        [info] Set current project to castillo (in build file:/tmp/scala_buildpack_build_dir/)
remote:        [info] Compiling 4 Scala sources and 1 Java source to /tmp/scala_buildpack_build_dir/target/scala-2.10/classes...
remote:        [success] Total time: 24 s, completed Jan 4, 2015 5:22:51 PM
remote:        [info] Wrote /tmp/scala_buildpack_build_dir/target/scala-2.10/castillo_2.10-1.0-SNAPSHOT.pom
remote:        [info] Packaging /tmp/scala_buildpack_build_dir/target/castillo-1.0-SNAPSHOT-assets.jar ...
remote:        [info] Packaging /tmp/scala_buildpack_build_dir/target/scala-2.10/castillo_2.10-1.0-SNAPSHOT.jar ...
remote:        [info] Done packaging.
remote:        [info] Done packaging.
remote:        [success] Total time: 1 s, completed Jan 4, 2015 5:22:51 PM
remote: -----> Dropping ivy cache from the slug
remote: -----> Dropping sbt boot dir from the slug
remote: -----> Dropping compilation artifacts from the slug
remote: -----> Discovering process types
remote:        Procfile declares types            -> (none)
remote:        Default types for Play 2.x - Scala -> web
remote: 
remote: -----> Compressing... done, 71.8MB
remote: -----> Launching... done, v6
remote:        https://stormy-crag-3594.herokuapp.com/ deployed to Heroku
remote: 
remote: Verifying deploy... done.
To https://git.heroku.com/stormy-crag-3594.git
 * [new branch]      master -> master
{%endhighlight%}

This last couple of lines show that the Play application has been built and published. It is available at 
[https://stormy-crag-3594.herokuapp.com](https://stormy-crag-3594.herokuapp.com). Examining the build output 
we see that we are using the wrong versions on the Heroku instance: Scala 2.10 rather than 2.11.4 and Play 2.3.4 
rather than 2.3.7. 

## Instrumentation to monitor the logs

## Instrumentation for basic application monitoring

## Refactor to a basic multi-module project

