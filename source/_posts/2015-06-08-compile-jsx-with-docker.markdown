---
layout: post
title: "Compile JSX with Docker"
date: 2015-06-08 12:45:51 -0400
comments: true
categories: docker javascript web
---

Users of [ReactJS](https://facebook.github.io/react/index.html) may be familiar with [JSX](https://facebook.github.io/react/docs/jsx-in-depth.html), "a Javascript syntax extension that looks similar to XML." JSX makes writing React components a little bit easier. I wrote a bunch of JSX components when I built [Refer Madness](https://www.refer-madness.com), and I wanted to find a sane way to compile my JSX files into plain JS files. As I was already using Docker to run my server-side and my [compile my SASS](/blog/2015/05/17/compile-sass-with-docker/), I created [a Docker image to compile my JSX code](https://registry.hub.docker.com/u/larryprice/jsx/) whenever file changes were detected.

I have a simple JSX file:

``` javascript login.jsx
var LoginButton = React.createClass({
  togglePanel: function() {
    if (window.location.pathname === "/") {
      $(".title").toggleClass("shrink fast");
    }
    $("#authenticate-panel").collapse('toggle');
  },
  render: function() {
    return (
      <div className="col-xs-12 col-sm-2 text-center">
        <button className="login-btn btn btn-default" data-toggle="collapse" onClick={this.togglePanel}
                aria-expanded="false" aria-controls="authenticate-panel">
          <span className="glyphicon glyphicon-lock"></span>
          Sign Up or Log In
        </button>
      </div>
    )
  }
});
```

With [Docker already installed](https://docs.docker.com/installation/#installation), I fire up a new container:

``` bash
$ docker run -v $PWD:/src --rm larryprice/jsx
```

In this case, `$PWD` is the directory where my JSX files live. The `-v` flag creates a volume in my container. The `--rm` flag will remove the container when I'm finished using it (for convenience). `larryprice/jsx` is the image I want to use. This command will watch the current directory and compile the JSX whenever it detects a file change. The generated files will be created next to the JSX files.

What's in the image, you ask? It's just the `node:latest` image with `react-tools` installed which automatically runs the command `jsx --extension jsx --watch src/ src/`. It's fairly simple, but way easier than having to do all this myself.

If I want to use the JSX image with docker-compose alongside the rest of a web application, I might have something like this:

``` yaml docker-compose.yml
app:
  image: golang:1.4
  working_dir: /go/src/simple-golang-app
  command: go run main.go
  volumes:
    - ./simple-golang-app:/go/src/simple-golang-app
jsx:
  image: larryprice/jsx
  volumes:
    - ./public/scripts:/src
```

It means not having to install Node or npm on your system if you don't already have it. Being able to compile my SCSS, JSX, and Go code all in once command was a lifesaver, and being able to recreate this environment anywhere meant keeping my computers in-sync was simple. Life, love, and Docker.
