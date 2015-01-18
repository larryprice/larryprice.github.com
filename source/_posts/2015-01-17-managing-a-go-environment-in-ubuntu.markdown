---
layout: post
title: "Managing a Go Environment in Ubuntu"
date: 2015-01-18 10:58:04 -0500
comments: true
categories: golang linux
---

Many moons ago, I wrote about [setting up a Go environment in Ubuntu](/blog/2013/12/15/setting-up-a-go-environment-in-ubuntu-12-dot-04). After writing that post, I dropped Go development for nearly a year. Today I run the [Indy Golang meetup](http://www.meetup.com/Indy-Golang/events/219612982/), and soon I'll be starting a new work project where I'll be recommending a Go-based tech stack. I've learned a thing or two about Go and managing its dependencies since I wrote that initial blog post, and I intend to give a short presentation to my meetup about my recent findings. Before I do, I thought I'd write a preliminary blog post detailing the tools I use to keep my Go environment sane.

###Installing Go###

The easiest way to install Go in Ubuntu is through `aptitude`. However, the default version in the Ubuntu repos gets stale fast. I found a tool similar to `rvm` for downloading and installing local versions of Go called [gvm](https://github.com/moovweb/gvm). For better or worse, `gvm` is installed through a bash script. Fortunately, it doesn't require sudo:

``` bash
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

At this point I removed all my previous finaglig with `$GOROOT` and `$GOPATH` from my dot files. You can use `gvm listall` to see all available versions of Go. As of the writing of this blog post, `go1.4.1` is the latest release; however, `go1.4` is the most recent release available with `gvm`. I'm not sure when the list is updated, but I have confidence that it is fairly regular. To install a Go and set it as the default Go:

```bash
$ gvm install go1.4 --default
$ which go
/home/lrp/.gvm/gos/go1.4/bin/go
```

###Managing Packages###

This is where I come to a fork in the road: `gvm`, the tool we used to install the desired version of Go, has a concept of pkgset similar to `rvm gemset`. However, I find the syntax for using a pkgset tiresome every time I come into a directory. I prefer something more automatic. As an additional pain, `gvm` does not provide a mechanism for installing dependencies from a list of known dependencies. I sought out other tools to address these pains and found [gpm](https://github.com/pote/gpm) and [gvp](https://github.com/pote/gvp).

`gpm` is a tool used to manage Go packages. It reads in a file called `Godeps` which contains a list of packages with versions and can install them from their individual sources. I'm currently infatuated with `gpm` as it addresses a lot of concerns I had when initially learning Go: shared local dependencies, unclear versioning, and installing dependencies from a fresh clone. Install `gvm` and `gvp`:

``` bash
$ pushd /tmp
$ git clone https://github.com/pote/gvp.git && cd gvp
$ ./configure && sudo make install
$ cd /tmp
$ git clone https://github.com/pote/gpm.git && cd gpm
$ ./configure && sudo make install
$ popd
```

We can follow two paths here: using `gvm pkgset` or using a local `.godeps` directory to store our dependencies discretely. For these examples, I'll create a directory called `gotest` with a single file in it:

``` go hello.go
package main

import "github.com/go-martini/martini"

func main() {
    server := martini.Classic()
    server.Get("/", func() string {
        return "<h1>Hello, world!</h1>"
    })

    server.Run()
}
```

####Method 1: gvm pkgset alongside gpm####

Create and start using a new pkgset:

``` bash
$ gvm pkgset create gotest
$ echo $GOPATH
/home/lrp/.gvm/pkgsets/go1.4/global
$ gvm pkgset use gotest
$ echo $GOPATH
/home/lrp/.gvm/pkgsets/go1.4/gotest:/home/lrp/.gvm/pkgsets/go1.4/global
```

What this means is that we are now using our `gotest` pkgset as default, but the global pkgset will be used to dig up any missing packages. In order to install any dependencies, we need to create a `Godeps` file for `gpm` to consume. Our application from above has a dependency on `go-martini`, so let's add a dependency to `v1.0` in our Godeps file:

``` text Godeps
github.com/go-martini/martini v1.0
```

After you run `go build` to verify that the dependencies aren't installed, run `gpm install` to pull the required packages into your specified pkgset. Run `go build` again and revel in your own brilliance.

That was pretty great, right? The only issue is remembering to type `gvm pkgset use gotest` every time you restart your terminal or switch projects. Otherwise, `gvm` is practically a replacement for one of my favorite ruby tools `rvm`.

####Method 2: gpm and gvp####

`gpm` is intended to be similar to `npm`, a package management tool for NodeJS. The author suggests using a tool called `gvp` to set the `GOPATH` without much thought. If we start a fresh terminal in our example directory, we can use `gvp` to set up our `GOPATH`:

``` bash
$ echo $GOPATH
/home/lrp/.gvm/pkgsets/go1.4/global
$ source gvp
$ echo $GOPATH
/home/lrp/Projects/2015/gotest/.godeps:/home/lrp/Projects/2015/gotest
```

You can note the difference in our `GOPATH` here relative to `gvm`: we will be using our current directory as a source of code in addition to a local `.godeps` directory. Go ahead and add `.godeps` to your `.gitignore` file or equivalent. We can use the same `Godeps` file as before:

``` text Godeps
github.com/go-martini/martini v1.0
```

We run `gpm install` to install the dependencies to our local `.godeps` directory.

I prefer this method to using pkgsets. I've had better luck building projects with complicated structures, and it's a lot easier for me to run `source gvp` than it is to remember the name of my pkgset. Both methods work pretty well and give me warm fuzzies about managing my dependencies. I'm certain that as Go continues to mature more solutions will come available. I've also been researching using Docker with gpm only, which requires very little tweaking to what I've already discussed here.
