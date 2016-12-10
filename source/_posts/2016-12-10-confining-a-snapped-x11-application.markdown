---
layout: post
title: "Confining a Snapped X11 Application"
date: 2016-12-10 17:30:38 -0500
comments: true
categories: ubuntu snappy
---

_Looking for the code? Look no further: [https://github.com/larryprice/pingus-snap](https://github.com/larryprice/pingus-snap)_

In my [last post](/blog/2016/12/07/creating-a-snap-from-existing-deb-packages/), I demonstrated creating a snap package for an application available in the archive. I left that application unconfined, which is taboo in the long run if we want our system to be secure. In a few steps, we can add the necessary components to confine our `pingus` snap.

For reference, this is the original `snapcraft.yaml` file for creating a `pingus` snap, except that we've updated the `confinement` property to `strict`:

``` yaml snapcraft.yaml
name: pingus
version: '0.1'
summary: Free Lemmings(TM) clone
description: |
    Pingus is a free clone of the popular Lemmings game.
    |
    Your goal is to guide a horde of penguins through a world full of obstacles
    and penguin traps to safety. Although penguins (unlike lemmings) are rather
    smart, they sometimes lack the necessary overview and now rely on you to
    save them.

grade: devel
confinement: strict

parts:
  archives:
    plugin: nil
    stage-packages:
      - pingus
  env:
    plugin: dump
    organize:
      pingus.wrapper: usr/bin/pingus

apps:
  pingus:
    command: pingus
```

If you're feeling bold, you can build and install the snap from here, but be warned that this led me into an ncurses nightmare that I had to forcibly kill. That's largely because `pingus` depends on X11, which is not available out-of-the-box once we've confined our snap. If we want to use X11, we're going to need to connect to it using the snap-land concept of [interfaces](http://snapcraft.io/docs/core/interfaces). Interfaces allow us to access shared resources and connections provided by the system or other snaps. There's some terminology to grapple with here, but the bottom line is that a "slot" provides an interface which a "plug" connects to. You can see a big list of available interfaces with descriptions [on the wiki](http://snapcraft.io/docs/reference/interfaces). Our `pingus` app will "plug" into the X11 interface's "slot":

``` yaml snapcraft.yaml
# ...
apps:
  pingus:
    command: pingus
    plugs:
      - x11
```

You can build and install the new snap with the `--dangerous` flag for your local confined snap. After that, you can verify the interface connection with the `snap interfaces` command:

``` bash
$ snapcraft
$ sudo snap install --dangerous pingus_0.1_amd64.snap
pingus 0.1 installed
$ snap interfaces
Slot                     Plug
:alsa                    -
# ...
:upower-observe          -
:x11                     pingus
```

Now, when we run `pingus`... it works! Well, video works. If you want sound, we'll also need the `pulseaudio` interface:

``` yaml snapcraft.yaml
# ...
apps:
  pingus:
    command: pingus
    plugs:
      - x11
      - pulseaudio
```

Once again: build, install, and run... et voilà! Is it just me, or was that surprisingly painless? Of course, not all applications live such isolated lives. Make note that the x11 interface is supposed to be a transitional interface, meaning that we would rather our app fully transition to Mir or some alternative. To go a step further with this snap, we could create a `snapcraft.yaml` to build from source to get the absolute latest version of our app. At this point, we can change our `grade` property to `stable` and feel good about something that we could push to the store for review.

_Any code you see here is free software. Find the project here: [https://github.com/larryprice/pingus-snap](https://github.com/larryprice/pingus-snap)_
