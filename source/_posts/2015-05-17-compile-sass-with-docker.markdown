---
layout: post
title: "Compile Sass with Docker"
date: 2015-05-17 16:11:18 -0400
comments: true
categories: sass web docker
---

[SASS](http://sass-lang.com/) is a delightful way to write CSS, but compiling SASS for your web application might not be the most obvious task. There are different installation methods, most of them involving cluttering up your filesystem with external dependencies.  Compiling SASS is easy when you're doing `rails` or `sinatra`-work, but I demand to write code in a compiled language. So I created [a docker image](https://registry.hub.docker.com/u/larryprice/sass/) to compile SASS code automatically called `larryprice/sass`.

Running this image is easy. Let's say I have some SASS code:

``` sass /home/larry/test-project/public/css/style.scss
.service-home .search-area {
  margin-top: 0;

  .service-name {
    font-size: 44px;
    font-weight: bold;
  }
}
```

Given that [I have Docker already installed](https://docs.docker.com/installation/#installation), I'll start up my Docker container:

``` bash
$ docker run -v /home/larry/test-project/public/css:/src --rm larryprice/sass
```

That's it. The directory `/home/larry/test-project/public/css` (containing my SASS code) is now being watched by the SASS compiler. Any time I make a change to a file in that directory, my code will automatically be recompiled. The above command runs the SASS watcher in the foreground, which allows me to see any compile errors in my terminal window. If I wanted to run the SASS compiler in the background, I just have to add the `-d` flag:

``` bash
$ docker run -v /home/larry/test-project/public/css:/src --rm -d larryprice/sass
```

If my project is using `docker-compose`, I could add an image in my `docker-compose.yml` to run the SASS compiler alongside my application. For instance, if I'm running a web application with `go`:

``` yaml docker-compose.yml
app:
  image: golang:1.4
  working_dir: /go/src/simple-golang-app
  command: go run main.go
  volumes:
    - ./simple-golang-app:/go/src/simple-golang-app
sass:
  image: larryprice/sass
  volumes:
    - ./public/css:/src
```

Now, running `docker-compose up` would also run the SASS compiler.

I like using `docker-compose` in these situations as it limits the number of conflicting tools installed on my host machine. If I ever decide that SASS is the worst or that ruby is the devil, all I have to do is `docker rmi larryprice/sass`. No need to run `find` commands for every file with `sass` in the title; just a single, simple command to destroy all evidence.
