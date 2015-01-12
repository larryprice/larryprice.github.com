---
layout: post
title: "An Example Use Case for Docker"
date: 2015-01-11 19:22:43 -0500
comments: true
categories: docker web ollert
---

I spent a lot of time last week asking questions about Docker. [What](#whatdocker) is Docker? How could Docker help me [day-to-day](#whydocker)? How easy is Docker to [use](#howdocker)? How does Docker like its eggs cooked? Isn't Docker a brand of sneakers?

###<a name="whatdocker"></a>What is Docker?###

[Docker](https://www.docker.com/) is a utility for maintaining system environments. Docker capitalizes on [Linux containers](https://en.wikipedia.org/wiki/LXC), a method of operating system virtualization which isolates multiple process groups on a single host.

Through a series of commands, Docker pulls up a base system image and applies changes to create a custom image. Docker provides the means to access any of these step-level containers for further manipulation. Docker uses a unique layered system such that sibling layers can utilize the same base images, in contrast to a virtual machine which would require multiple copies of things like operating systems, shared libraries, and shared binaries. The image below is an excellent visualization of the difference between a virtual machine and a Docker container.

![VM vs LXC](http://i.imgur.com/Lps6K6y.png?1)

###<a name="whydocker"></a>Why use Docker?###

Docker's primary job is to take in a series of commands and spit out a clean environment with those settings. This is especially useful in deployment. I can take the Docker `ubuntu` image, download and install all my dependencies, copy over my application code, and run my application given some environment variables.

All of that should sound somewhat familiar if you've ever used [heroku](https://heroku.com). Heroku performs very similar tasks to get your application up and running: take a Linux image, download base tools for ruby/python/nodejs/whatever, install application-specific dependencies (through bundler, flask, npm, etc.), and run your application given some environment. Docker gives you the power of heroku at the development level. ...Sort of.

If I can create a Docker container for my application to run in, I can set up a build server to use a container to run my tests. I can use that same container to build a _clean_ staging environment. I can use the staging container to build an _identically clean_ production environment. With that knowledge in hand, I know the exact state of the production environment every time I deploy and I can reproduce it locally.

Theoretically, I can even use Docker for setting up a development environment, although after a few days of attempting this I still think you're better off running natively.

Of course, Docker keeps [a big list of examples](https://www.docker.com/resources/usecases/) from big-name company use-cases if you're interested.

###<a name="howdocker"></a>Example Usage###

Brass. Tacks. Let us get down to them, compadre.

You probably want to [install Docker](https://docs.docker.com/installation/#installation) first. If you're not using Linux, have fun installing [boot2docker](https://github.com/boot2docker/boot2docker), the rest of us are going to get started without you.

I started by trying to bootstrap my environment for [Ollert](https://ollertapp.com) with Docker. Ollert uses ruby-2.2, QtWebkit (in test), and MongoDB. It uses bundler to install any required ruby gems. Not too complicated, but I've noticed it's never easy to get a new developer's environment quite right.

We start out with the official [ruby:2.2.0](https://registry.hub.docker.com/_/ruby/) image from the Docker Hub:

``` bash
$ docker run ruby:2.2.0 echo "B-b-b-b-brass t-t-t-t-tacks!"
```

OMG that step will take forever if you've never downloaded the base `debian` image. It downloads and sets up quite a few layers. If you're interested in what it's doing behind the scenes and you can read Dockerfiles, [this file](https://github.com/docker-library/ruby/blob/b7fefd2fa79882da90feb0718430680c77c5fa8b/2.2/Dockerfile) is what's being executed. Anyway, when it's done you should see a friendly reminder about what we've gotten down to. We use `docker run` to run (download first if necessary) an image; in this case, the `ruby:2.2.0` image. Everything after the image name is the command to run. Now that we've downloaded some base images, you can check out your available images using `docker images`.

Now I need to install my system-level dependencies:

``` bash
$ docker run ruby:2.2.0 apt-get update
```

Note how this time the base image was already found in your local repository, resulting in a command that ran pretty quickly (based on your internet speeds (sorry Comcast customers!)). But what have we really done so far? We've created two separate containers: one with our initial echo command (useless) and one with all our updates. To see these containers, use `docker ps -a`. This will give you output similar to the following:

``` bash
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS             NAMES
ad5ddd55f2c2        ruby:2.2.0        "apt-get update"         2 seconds ago       Exited (0) 2 seconds ago                        mad_curie
10cbaac4488c        ruby:2.2.0        "echo 'B-b-b-b-brass"    15 minutes ago      Exited (0) 15 minutes ago                       mad_perlman
```

These are now the containers available. We can create a new image from the first container using `docker commit ad5 rubyapp`, which will allow us to use it to create further containers. However, if we were to do this for every command we wanted to execute, we might be here for a while. We could go into `bash` on the base image and do all of our steps:

``` bash
$ docker run -it ruby:2.2.0 /bin/bash
root@1be43510341e:/# apt-get update
...
root@1be43510341e:/# apt-get auto-remove
...
root@1be43510341e:/# apt-get install -y --force-yes libqtwebkit-dev mongodb
...
```

We could then use this image to run our app - however, this is also tedious and a bad solution. We want something that we can see on a granular level and reproduce every time for a base image. Fortunately, Docker provides us an easy way to do this using a DSL. Introducing the `Dockerfile`:

``` Dockerfile
# base image
FROM ruby:2.2.0

#install system-level dependencies
RUN apt-get update && apt-get autoremove -y && apt-get install -y --force-yes libqtwebkit-dev mongodb

# install gems from /tmp such that bundling is CACHED
WORKDIR /tmp
ADD Gemfile Gemfile
ADD Gemfile.lock Gemfile.lock
ADD .env .env
RUN bundle install

# load application source
ADD . /usr/src/app
WORKDIR /usr/src/app

# port where application is served
EXPOSE 5000
```

The syntax is a little different, but all we're doing is telling Docker our base image, issuing commands, and copying files. The `ADD` command allows us to copy files from our host system. In this case, I copy over `.` to `/usr/src/app` in the container. I also copy over my Gemfile separately to [cache the bundle so it does not install every time](http://ilikestuffblog.com/2014/01/06/how-to-skip-bundle-install-when-deploying-a-rails-app-to-docker/). I then expose the port I want my application to use. Run this file as such:

``` bash
$ docker build -t rubyapp .
```

This creates an image called `rubyapp` and a container for every line of the Dockerfile that is run. Although your first build may take a moment, subsequent builds will be cached and should be significantly faster. Now, if we want to run my application:

``` bash
$ docker run -d --name rubyappinstance rubyapp foreman start -d /usr/src/app
```

I use `foreman` to start my application from the given directory. I tell Docker that the application will be daemonized using the `-d` flag. If I check my running containers with `docker ps`, I'll see my application running. If I want to stop it, I just run `docker stop rubyappinstance`.

I'm going to stop there for now. In order to get Ollert working properly, I also need to [link a Mongo database](http://docs.docker.com/userguide/dockerlinks/) and change some environment variables in my application, but those are relatively easy tasks.

###Is it worth it?###

The only conclusion is a definite maybe. Docker is definitely pretty cool. It may be able to help you deploy custom applications easier; for Ollert, it feels like overkill. There is a lot of overhead in downloading core versions of different operating systems, and I already find myself itching to clean up all the leftover Docker images/containers on my machine I used once and never again. After getting the Docker development out of the way (building and testing a Dockerfile), you may save yourself some time in the future if you have to change hosting services or CI environments. Try it out! It's a pretty neat concept and definitely worth your attention in 2015.
