---
layout: post
title: "Quick Start to Vendor Go Dependencies with govendor"
date: 2018-04-26 19:34:04 -0500
comments: true
categories: go golang
---

I recently spent a few days adapting my [Go for Web Development video series](https://www.packtpub.com/web-development/go-web-development-video) into a text-based course. In doing so, I had the chance to investigate some of the new vendoring tools available in Go. As of Go 1.5, "vendoring" dependencies has become the norm. Vendoring means tracking your dependencies and their versions and including those dependencies as part of your project.

In particular, I explored the uses of the [govendor](https://github.com/kardianos/govendor) package, mostly because it's supported by default by Heroku. The docs on the GitHub are a lot more thorough than what I'll go over here.

`govendor` is easily installed within the go ecosystem. Assuming that `$GOPATH/bin` is in your path:

``` bash
$ go get -u github.com/kardianos/govendor
$ which govendor
/home/lrp/go/bin/govendor
```

Now we just initialize the `govendor` directory and start installing dependencies. The `govendor fetch` command is pretty much all you'll need:

``` bash
$ govendor init
$ govendor fetch github.com/jinzhu/gorm
$ govendor fetch golang.org/x/crypto/bcrypt
```

`init` will create a `vendor` directory in your project path. Go will check this directory for any packages as though they were in your `$GOPATH/src` directory. The `fetch` calls will add new packages or update the given package in your `vendor` directory; in this case, I've fetched the latest versions of `gorm` and `bcrypt`.

This might seem painful, but the thing to do next is to commit everything in the vendor directory to your repository. Now you have it forever! This means that anyone who wants to run this version of your code in the future doesn't have to worry about dependency versions and can instantly run your package with a valid go install.

If you don't want to add all these packages to your repository, I don't blame you. You can get around this by committing just your `vendor/vendor.json` file and then using `govendor sync` to install the missing packages after downloading your source code. This should be familiar to anyone who's used `bundler` in ruby, `virtualenv` in python, or `npm` in Node.JS. If you're using git, you'll want a `.gitignore` with the following:

```
vendor/*
!vendor/vendor.json
```

This will ignore everything in `vendor/` except for the `vendor.json` file which lists all your packages and their corresponding versions. Now, to install any packages from `vendor.json` that you don't already have in your `vendor` directory:

``` bash
$ govendor sync
```

`govendor` is a pretty powerful tool for vendoring your go dependencies and getting your application Heroku-ready, and I recommend [checking out the docs](https://github.com/kardianos/govendor) for a more advanced overview. There are also many other vendoring options available, including an official go vendoring tool called [dep](https://github.com/golang/dep) that works with go 1.9+. `dep` will most definitely play a big role in refining the ideas that these third-party tools have created and the go ecosystem will become more stable.
