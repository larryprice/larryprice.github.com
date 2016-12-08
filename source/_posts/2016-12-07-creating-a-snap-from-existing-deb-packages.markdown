---
layout: post
title: "Building Snaps from Archived Packages"
date: 2016-12-07 22:10:08 -0500
comments: true
categories: ubuntu snappy
---

If you haven't heard, [snaps](//snapcraft.io) are a new, modern packaging format made by the guys at Ubuntu. Snaps give every app a confined environment to live in, making desktops more secure and dependencies less of a hassle. One common way to create a snap is to simply use existing packages from the Ubuntu archives.

Let's try to create a snap for the game [pingus](https://lgdb.org/game/pingus). `pingus` is a great little Lemmings clone that we can easily convert to a snap. We'll start by installing the necessary dependencies for snap building (see [the snapcraft website](//snapcraft.io/create/) for more):

``` bash
$ sudo apt install snapcraft
```

Now we can initialize a project directory with snapcraft:

``` bash
$ mkdir -p pingus-snap && cd pingus-snap
$ snapcraft init
```

`snapcraft init` creates the following sample file to give us an idea of what we'll need to provide.

``` yaml snapcraft.yaml
name: my-snap-name # you probably want to 'snapcraft register <name>'
version: '0.1' # just for humans, typically '1.2+git' or '1.3.2'
summary: Single-line elevator pitch for your amazing snap # 79 char long summary
description: |
  This is my-snap's description. You have a paragraph or two to tell the
  most important story about your snap. Keep it under 100 words though,
  we live in tweetspace and your description wants to look good in the snap
  store.

grade: devel # must be 'stable' to release into candidate/stable channels
confinement: devmode # use 'strict' once you have the right plugs and slots

parts:
  my-part:
    # See 'snapcraft plugins'
    plugin: nil
```

Most of these values for our `pingus` snap should be obvious. The interesting markup here is in `parts`, which is where we'll describe how to build our snap. We'll start by taking advantage of the `nil` plugin to simply unpack the `pingus` deb from the archive. We define our list of debs to install in a list called `stage-packages`. We'll also define another section, `apps`, to tell `snapcraft` what binaries we want to be able to execute. In our case, this will just be the `pingus` command. Here's what my first draft looks like:

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
confinement: devmode

parts:
  archives:
    plugin: nil
    stage-packages:
      - pingus

apps:
  pingus:
    command: usr/games/pingus
```

Nice, right? Building and installing our snap is easy:

``` bash
$ snapcraft
$ sudo snap install --devmode pingus_0.1_amd64.snap
pingus 0.1 installed
```

We used `devmode` here because our app will be running unconfined (a topic for another blog post). Now, for the moment of truth! The snap tools automatically put our new app in `PATH`, so we can just run `pingus`:

``` bash
$ pingus
/snap/pingus/x2/usr/games/pingus: 2: exec: /usr/lib/games/pingus/pingus: not found
```

¡Ay, caramba! We've run into a fairly common issue while snapping legacy software: __hardcoded paths__. Fortunately, the corresponding `pingus` executable is very simple. It's trying to execute a command living in `/usr/lib/games/pingus`, which is not in our snap's `PATH`. The easiest way to fix this is to fix the `pingus` executable. Since we don't want to spend time modifying the upstream to use a relative path, we can create our own version of the `pingus` wrapper locally and copy it into our snap. The only change to this new wrapper will be prepending the snap's install path `$SNAP` to the absolute paths:

``` bash pingus.wrapper
#!/bin/sh
exec $SNAP/usr/lib/games/pingus/pingus --datadir $SNAP/usr/share/games/pingus/data $@
```

Now we can update our yaml file with a new part called `env` which will use the `dump` plugin to copy our wrapper file into the snap. We'll also update our command to call the wrapper:

``` yaml snapcraft.yaml
# ...

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

When you run `snapcraft` this time, the `env` part will be built. After performing another install, you can run `pingus`, and you should be greeted with one of the best Lemmings clones available! Because we're running unconfined in devmode, this all just works without any issues. I intend to write another blog post in the near future with the details on confining `pingus`, so look out for that soon. I may also go into detail on building more complex cases, such as building snaps from source and building custom plugins, or reviewing a case study such as the `libertine` snap.

For much, much more on snaps, be sure to visit [snapcraft.io](//snapcraft.io/create/). If you're looking for a published version of pingus as a snap, you can try `sudo snap install --devmode --beta pingus-game`, and you can run the game with `pingus-game.pingus`.

_Source code available at [https://github.com/larryprice/pingus-snap](https://github.com/larryprice/pingus-snap)._
