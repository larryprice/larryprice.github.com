---
layout: post
title: "Writing Go/QML Convergent Ubuntu Apps"
date: 2016-07-07 16:31:23 -0400
comments: true
categories: golang ubuntu
---

Thinking about writing an Ubuntu application that will work in Unity 8? You'll be writing a "convergent" app, which is an application that can respond to touch or mouse, and will adapt appropriately to a phone, tablet, or desktop screen. It will even be able to update its display on-the-fly if, say, you plug your phone into a monitor or bluetooth keyboard.

There are _lots_ of ways to write a convergent application with the [Ubuntu SDK](https://developer.ubuntu.com/en/phone/platform/sdk/). You can build apps in pure QML, C++ with QML, QtQuick, pure HTML5/Javascript, or even __Golang with QML__. We'll be focusing on go apps for the rest of this post.

Go is a young language, and the recent release of [go 1.6](https://blog.golang.org/go1.6) has introduced some nasty changes particularly involving cgo. It has also introduced vendoring by default, which is a welcome change.

I'm using go 1.6.2. For me, the current project template provided by the Ubuntu SDK feels quite broken. I can't get things to build locally, and furthermore I see no options for pushing an ARM build to my device.

Fortunately, I found this [ubuntu-go-qml-template](https://github.com/nikwen/ubuntu-go-qml-template) which is a template to enable running/building a go/qml application locally while also supporting building/installing onto an arm device. The kicker? This tool was designed to work for go 1.3.3. Sigh! Since I'm unwilling to compromise and use old technology, I [forked the project](https://github.com/larryprice/ubuntu-go-qml-template) and updated it to fit my more modern needs.

Because our go template will depend on QML, we depend on the [go-qml](https://github.com/go-qml/qml) project to create the necessary C bindings to allow us to use QML. However, with the update to go 1.6, the current version (revision 2ee7e5f) of go-qml will give a runtime error from cgo with the wonderfully helpful `panic: runtime error: cgo argument has Go pointer to Go pointer`. Another thorn in our side. Fortunately, this issue is in previously tread-upon ground and there is a [fork of go-qml](https://github.com/sjb/qml/tree/go1.6-port) with enough of the cgo issues fixed to run our application with no problems. In the `ubuntu-go-qml-template` I forked above, I've gone ahead and vendored this fork of `go-qml`. It all works because of the default vendoring available in go 1.6.

With that background out of the way, let's run through getting a project started:

``` bash
$ sudo apt-get install golang g++ qtdeclarative5-dev qtbase5-private-dev \
                       qtdeclarative5-private-dev libqt5opengl5-dev \
											 qtdeclarative5-qtquick2-plugin
# installing dependencies...
$ git clone https://github.com/larryprice/ubuntu-go-qml-template.git your-project-name
# cloning repo...
$ cd your-project-name
$ chroot-scripts/setup-chroot.sh
# building a chroot for ubuntu-sdk-15.04
# distro=vivid, arch=arm
# with go 1.6.2 with armhf compilation support
```

You may need to have other dependencies including `click` or `phablet-tools`. The above commands installed dependencies, cloned the repo, and built and/or updated a chroot for building our source for arm.

Next you'll want to setup the project for your own needs. The original template creator included a nifty `setup` script to get this done:

``` bash
$ ruby setup.rb -v -n your-project-name -a "Your Name" -e "your.email@example.com" \
                   -d "your-developer-namespace"
```

This will do some fancy gsub and file renaming to make the template your own.

If you check `src/`, you'll find a `main.go` file ready for your use. You'll also find a `main.qml` file in `share/qml/`. You can vendor all of your dependencies in `vendor/`, where you'll already find the `qml` package.

As far as getting your application to work, there are more scripts available:

```
$ ./build.sh
# this will build your project locally
$ ./run.sh
# this will build your project locally and then run it on the desktop
```

The best part about this template is the ability to build for arm and load your applications onto a device:

```
$ ./build-in-chroot.sh
# builds the package using the vivid+armhf chroot we set up previously
$ ./install-on-device.sh
# builds for vivid+armhf and installs the click directly on
# the first USB-connected Ubuntu Touch device
```

Now you can run and install go/qml applications on desktop or on devices. Time to go build something cool!

_Disclaimer: In the future, this template will likely need to be updated for new go versions or new default versions of the Ubuntu SDK. Don't be afraid to make a comment below or submit a PR._
