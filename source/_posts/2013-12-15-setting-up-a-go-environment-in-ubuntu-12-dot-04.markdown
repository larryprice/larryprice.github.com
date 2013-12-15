---
layout: post
title: "Setting up a Go environment in Ubuntu"
date: 2013-12-15 18:40
comments: true
categories: golang linux
---

Some very light cajoling led me to do some investigation into [Google Go](http://golang.org/) (often called __golang__ for ease of internet search). This is a brief recounting of how I got up and running on [Ubuntu](http://ubuntu.com) (first 12.04 and then 13.10). Luckily, this has been made espcially easy for us with the introduction of a [golang package in the Ubuntu package repositories](http://packages.ubuntu.com/precise/golang). There are also [official installation instructions](http://golang.org/doc/install) if you don't like mine.

Open up a terminal and let loose:

``` bash
$ sudo apt-get install golang
```

The download is pretty heavy, so this step may take some time. Eventually the installer for `golang-go` will ask you if you want to "Report installation of public packages to Go Dashboard." I've been choosing "No" to this question and have no complaints.

Now comes the fun part. Serious Go development relies on having a "workspace" setup involving a specific directory structure including `bin/`, `pkg/`, and `src/` directories. Google's [official set-up page](http://golang.org/doc/code.html) contains more information about these workspaces.

I'm not a big fan of putting a visible directory in `$HOME`, so I opted to make a hidden directory called `.go`. After creating the directory, the environment variable `$GOPATH` needs to be set and `$PATH` needs to be adjusted.

``` bash
$ mkdir ~/.go
$ echo "GOPATH=$HOME/.go" >> ~/.bashrc
$ echo "export GOPATH" >> ~/.bashrc
$ echo "PATH=\$PATH:\$GOPATH/bin # Add GOPATH/bin to PATH for scripting" >> ~/.bashrc
$ source ~/.bashrc
```

Now I'm going to create a go project and add a link to it in the workspace I just created.

``` bash
$ mkdir -p $GOPATH/src/github.com/user
$ mkdir ~/hello-go
$ ln -s ~/hello-go ~/.go/src/github.com/user/hello-go
```

For some actual test code, I'll add a file in my `hello-go/` directory called `hello.go` with the following code:

``` go hello.go
package main

import "fmt"

func main() {
  fmt.Println("Hello world")
}
```

Now I'm going to install the binary created from compiling this code into my `$GOPATH` to verify that my workspace is set up correctly, then I'll run it to behold the fruit of my efforts.

``` bash
$ go install github.com/user/hello-go
$ hello-go
Hello world
```

Installing is not necessary every time I want to test that my code compiles; running `go build` in the source directory will create a local executable that can be executed for incremental testing.

If you're interested in learning golang, I would recommend doing the [go tour](http://tour.golang.org/). It goes way beyond a trivial hello world program and gives you some insight into many coplex go concepts (I even posted [my solutions as a gist](https://gist.github.com/larryprice/7647808), if you're interested).
