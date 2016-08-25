---
layout: post
title: "Maintaining X Applications in Unity 8"
date: 2016-08-24 17:29:59 -0400
comments: true
categories: ubuntu unity
---

The release of Ubuntu 16.10 Yakkety Yak in the coming months will bring about the public release of Unity 8 as a pre-installed desktop session (though not as the default session). It's been a long time coming, and there's a lot of new features which will break older applications. Canonical has unveiled [snappy](http://snapcraft.io) as the preferred packaging system for Unity 8, but what about all those old deb packages?

There have been a few other good posts about X applications on Unity 8 including [this one on dogfooding](http://mhall119.com/2016/05/dogfooding-unity-8/), [this one on Ubuntu Touch](https://kylenubuntu.blogspot.no/2016/07/running-x-apps-on-ubuntu-devices.html), and [this one on how it works under the covers](https://bregmatter.wordpress.com/2016/07/04/x11-applications-and-unity-8/). This blog post is explicitly about Unity 8 on desktop using the Libertine CLI, though can be applied to most devices running Ubuntu Touch.

_Disclaimer: I work for Canonical on one of the teams making all of this fancy stuff work._

### A (Very) Brief Explanation ###

The toolchain we'll be relying on is called `libertine`, and it's essentially a wrapper around unprivileged LXC and chroot-based containers. We prefer to use LXC containers on newer OSes, but we must continue supporting chroot containers on many devices due to kernel limitations.

### What You'll Need ###

For desktop Unity 8, you'll need the packages for `libertine`, `libertine-tools`, and `lxc` to get started. This will install a CLI and GUI for maintaining Libertine containers and applications.

If you're running Wily or newer, you can just run the following in your terminal:

``` bash
$ sudo apt install libertine
```

Otherwise, you'll need to add the stable overlay PPA first:

``` bash
$ sudo add-apt-repository ppa:ci-train-ppa-service/stable-phone-overlay
$ sudo apt-get update
$ sudo apt-get install libertine
```

### The GUI ###

At this point, if you're on desktop you can open up the GUI which will guide you through creating a new container and installing applications. Search the Dash (or Apps scope) for `libertine` and, given that we haven't pushed a buggy version recently, you'll be presented with a Qt application for maintaining containers. I highly recommend using the GUI, because then you are guaranteed not to be following out-of-date console commands.

...But maybe you prefer the terminal. Or maybe you're secretly SSH'd into the target machine or Ubuntu Touch device and need to use the terminal. If so...

### The CLI ###

The CLI we'll be using is `libertine-container-manager`. It has a `manpage`, a `--help` option, and autocomplete to help you out in a jam.

The first thing you'll want to do is create a container. There are a lot of options, but to create an optimal container for your current machine you only need to specify the `id` and `name` parameters:

``` bash
$ libertine-container-manager create --id desktopapps --name "Desktop Applications"
```

A couple of things to note here: Your `id` must be unique and conform to the simple click name regex - this is what will identify your container on a system level. The `name` should be human-readable so you can easily identify what might be inside your container. If you don't specify a `name`, your `id` will be used. The CLI will likely ask you for a password to use in the container in case you ever need it. You can leave this blank if you're not concerned with that kind of thing.

At this point, a bunch of things should be happening in your terminal. This will pull a container image for your current distro and install all the requirements to get started maintaining and running X apps. This could take anywhere from a few minutes to the next hour depending on your network and disk speeds. Once you're done, you can use the `list` subcommand to list all installed containers (note you probably just have one at this point). If you ever want to delete your container, you can run `libertine-container-manager destroy -i desktopapps`.

Once that's finished, we can start installing apps. To find apps available, you can use the `search-cache` subcommand:

``` bash
$ libertine-container-manager search-cache --id desktopapps --search-string "office"
```

This will return a few strings from the apt-cache of the container with `id` "desktopapps" that match "office". Now, if you want to install "libreoffice":

``` bash
$ libertine-container-manager install-package --id desktopapps --package libreoffice
```

This will install the full libreoffice suite. Nice! Similarly, you can use the `remove-package` subcommand to remove applications. Don't remember what apps you've installed? Use the `list-apps` command:

``` bash
$ libertine-container-manager list-apps --id desktopapps
```

Maybe you're an avid Steam for Linux gamer and want to try to get some games working. Since Steam still only comes in a 32-bit binary, you'll need to enable the multiarch repos, and then you can just install Steam like any other app:

``` bash
$ libertine-container-manager configure --id desktopapps --multiarch enable
...
$ libertine-container-manager install-package --id desktopapps --package steam
```

Steam will ask you to agree to their user agreement from the command line, which you should be able to do easily. If you need to use the `readline` frontend for dpkg, you can append `--readline` to the `install-package` command to enable it.

There are many other commands to explore to maintain your container, but for now I'll let you check the manpage or open the GUI to explore further.

### Running Apps ###

Now that you've installed some apps, you probably want to run them. You can install the Libertine Scope, which will allow you to peruse your installed apps in a Unity 8 session. You can either install it from the App Store on a device (search for "Desktop Apps Scope") or through `apt` on desktop with:

``` bash
$ sudo apt install libertine-scope
```

In a Unity 8 session, you can now find the scope and click on apps to run them. Note that there are many apps which still don't work, such as those requiring a terminal or `sudo`; consider these a work in progress.

### The Future ###

I've been toiling away the past few weeks getting a scope ready which can be used explicitly to install/remove X apps in Unity 8, like the current Ubuntu Software Center (or app store on Touch devices). This scope should be available anywhere the libertine scope is available, meaning that it will alleviate a lot of the pain associated with installing/removing apps for a large chunk of users. Using the Libertine GUI or Libertine CLI will still allow for much more customization, but those tools are largely designed with power users in mind.

Are you able to get libertine working on your system? Can you launch X applications to your heart's content? Let me know in the comments!
