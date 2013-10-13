---
layout: post
title: "Installing GTest and GMock Libs in Ubuntu 13.04"
date: 2013-10-13 11:02
comments: true
categories: ubuntu gtest gmock
---

I started trying to work on an open-source project and found that I needed to install [googletest](https://code.google.com/p/googletest/) and [googlemock](https://code.google.com/p/googlemock/) on my home machine. Seemed easy enough, I found a package called `google-mock` in the [Raring](http://packages.ubuntu.com/raring/google-mock) repositories which depends on a package called `libgtest-dev`. So I install it:

``` bash
$ sudo apt-get install -y google-mock
```

And the CMake file I was running before tells me that I have gmock, but I'm still missing gtest. What's going on here?

Well, there was a changeset applied in Ubuntu 12.04 (Precise) with the following text ([source](http://www.ubuntuupdates.org/package/core/precise/universe/base/gtest)):

>   Stop distributing static library (although still build it, to ensure gtest 
>   works). Upstream recommends against shipping the libary at all, just the 
>   source. (See: http://code.google.com/p/googletest/wiki/FAQ) 
>   The Debian maintainer plans to do this also (see BTS: 639795); do it in 
>   Ubuntu now to fulfil MIR requirements.
>
> -- Christopher James Halse Rogers Thu, 08 Mar 2012 17:45:29 +1100

What does that mean? That means we get to build and "install" the gtest libs ourselves. The source is conveniently installed in `/usr/src` after installing `libgtest-dev` (which we automatically got when we installed `google-mock`).

``` bash Installing gtest libs
$ sudo apt-get install -y cmake --quiet
$ cd /usr/src/gtest
$ sudo cmake -E make_directory build
$ sudo cmake -E chdir build cmake .. >> /dev/null
$ sudo cmake --build build >> /dev/null
$ ls build/libgtest*
build/libgtest.a  build/libgtest_main.a
$ sudo cp build/libgtest* /usr/local/lib/
$ sudo rm -rf build
```

The `>> /dev/null` can be dropped if you would like to see the output of these commands when successful, any error text will still show up with this redirect in place. I like to move all my personally-compiled libraries (and includes) to `/usr/local`, but you could just as easily copy them over to `/usr/lib`. All of this could also be done in `/tmp` if you're so inclined.
