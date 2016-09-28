---
layout: post
title: "Clean package building with pbuilder"
date: 2016-09-27 21:50:00 -0400
comments: true
categories: ubuntu
---

Whether I'm adding dependencies, updating package names, or creating new package spins, I always have issues testing my debian packages. Something will work locally, only to fail on jenkins under a clean environment. Fortunately, there's a nifty tool called `pbuilder` that exists to help out in these situations. `pbuilder` uses a chroot to set up a clean environment to build packages, and can even be used to build packages for systems with architectures different from your own.

_Note: All code samples were originally written from a machine running Ubuntu 16.10 64-bit. Your mileage may vary._

### Clean builds for current distro ###

Given a typical debian-packaged project with a `debian` directory (`control`, `rules`, `.install`), you can use `debuild` to build a package from your local environment:

``` bash
$ cd my-project
$ debuild
...
$ ls ../*.deb
my-project.deb
```

This works pretty well for sanity checks, but sometimes knowing your sane just isn't quite enough. My development environment is filled with libraries and files installed in all kinds of weird ways and in all kinds of strange places, so there's a good chance packages built successfully on my machine may not work on everyone's machine. To solve this, I can install `pbuilder` and set up my first chroot:

``` bash
$ # install pbuilder and its dependencies
$ sudo apt-get install pbuilder debootstrap devscripts
$ # create a chroot for your current distro with build-essential pre-installed
$ sudo pbuilder create --debootstrapopts --variant=buildd
```

Since I use `debuild` pretty frequently, I also rely on `pdebuild` which performs `debuild` inside of the clean chroot environment, temporarily installing the needed dependencies listed in the `control` file.

``` bash
$ cd my-project
$ pdebuild
$ ls /var/cache/pbuilder/result/*.deb
my-project.deb
```

Alternatively, I could create the `.dsc` file and then use `pbuilder` to create the package from there:

``` bash
$ # generate a dsc file however you like
$ cd my-project
$ bzr-builddeb -- -us -uc
$ cd ..
$ # use pbuilder to create package
$ sudo pbuilder build my-project.dsc
$ ls /var/cache/pbuilder/result/*.deb
my-project.deb
```

### Clean cross builds ###

Let's say that you need to build for an older distribution of Ubuntu on a weird architecture. For this example, let's say `vivid` with `armhf`. We can use `pbuilder-dist` to verify and build our packages for other distros and architectures:

``` bash
$ # create the chroot, once again with build-essential pre-installed
$ pbuilder-dist vivid armhf create --debootstrapopts --variant=buildd
$ # the above command could take a while, but once it's finished
$ # we can attempt to build our package using a .dsc file
$ pbuilder-dist vivid armhf build my-project-dsc
$ ls ~/pbuilder/vivid-armhf_result/*.deb
my-project.deb
```

### Custom, persistent chroot changes ###

In some cases, you may need to enable other archives or install custom software in your chroot. In the case of our vivid-armhf chroot, let's add the [stable-overlay ppa](https://launchpad.net/~ci-train-ppa-service/+archive/ubuntu/stable-phone-overlay) which updates the outdated vivid with some more modern versions of packages.

``` bash
$ # login to our vivid-armhf chroot, and save state when we're finished
$ # if --save-after-login is omitted, a throwaway chroot will be used
$ pbuilder vivid armhf login --save-after-login
(chroot) $ # install the package container add-apt-repository for convenience
(chroot) $ apt install software-properties-common
(chroot) $ add-apt-repository ppa:ci-train-ppa-service/stable-phone-overlay
(chroot) $ exit
$ # update packages in the chroot
$ pbuilder-dist vivid armhf update
```

`pbuilder` and chroots are powerful tools in the world of packaging and beyond. There are scripting utilities, as well as pre- and post-build hooks which can customize your builds. There are ways to speed up clean builds using local caches or other "cheats". You could use the throwaway terminal abilities to create and destroy tiny worlds as you please. All of this is very similar to the utility which comes from using docker and lxc, though the underlying "container" is quite a bit different. Using `pbuilder` seems to have a much lower threshold for setup, so I prefer it over docker for clean build environments, but I believe docker/lxc to be the better tool for managing the creation of consistent virtual environments.

_Further reading:_

[Pbuilder HowTo on the Ubuntu wiki](https://wiki.ubuntu.com/PbuilderHowto)
[Pbuilder tricks from the debian wiki](https://wiki.debian.org/PbuilderTricks)
