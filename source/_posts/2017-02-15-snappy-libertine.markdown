---
layout: post
title: "Snappy Libertine"
date: 2017-02-15 18:38:44 -0500
comments: true
categories: ubuntu snappy
---

Libertine is software suite for runnin X11 apps in non-X11 environments and installing deb-based applications on a system without dpkg. Snappy is a package management system to confine applications from one another. Wouldn't it be cool to run libertine as a snap?

Yes. Yes it would.

### snapd ###

The first thing to install is snapd itself. You can find installation instructions for many Linux distros at [snapcraft.io](https://snapcraft.io/docs/core/install), but here's the simple command if you're on a debian-based operating system:

``` bash
$ sudo apt install snapd
```

Ubuntu users may be surprised to find that snapd is already installed on their systems. snapd is the daemon for handling all things snappy: installing, removing, handling interface connections, etc.

### lxd ###

We use lxd as our container backend for libertine in the snap. lxd is essentially a layer on top of lxc to give a better user experience. Fortunately for us, lxd has a snap all ready to go. Unfortunately, the snap version of lxd is incompatible with the deb-based version, so you'll need to completely remove that before continuing. Skip this step if you never installed lxd:

``` bash
$ sudo apt remove --purge lxd lxd-client
$ sudo zpool destroy lxd                 # if you use zfs
$ sudo ip link set lxdbr0 down           # take down the bridge (lxdbr0 is the default)
$ sudo brctl delbr lxdbr0                # delete the bridge
```

For installing, in-depth instructions can be found [in this blog post by one of the lxd devs](https://stgraber.org/2016/10/17/lxd-snap-available/). In short, we're going to create a new group called `lxd`, add ourselves to it, and then add our own user ID and group ID to map to root within the container.

``` bash
$ sudo groupadd --system lxd                      # Create the group on your system
$ sudo usermod -G lxd -a $USER                    # Add the current user
$ newgrp lxd                                      # update current session with new group
$ echo root:`id --user ${USER}`:1 >> /etc/subuid  # Setup subuid to map correctly
$ echo root:`id --group ${USER}`:1 >> /etc/subgid # Setup subgid to map correctly
$ sudo snap install lxd                           # actually install the snap!
```

We also need to initialize lxd manually. For me, the defaults all work great. The important pieces here are setting up a new network bridge and a new filestore for lxd to use. You can optionally use zfs if you have it installed (`zfsutils-linux` should do it on Ubuntu). Generally, I just hit "return" as fast as the questions show up and everything turns out alright. If anything goes wrong, you may need to manually delete zpools, network bridges, or reinstall the lxd snap. No warranties here.

``` bash
$ sudo lxd init
Do you want to configure a new storage pool (yes/no) [default=yes]?
Name of the new storage pool [default=default]:
Name of the storage backend to use (dir or zfs) [default=zfs]:
Create a new ZFS pool (yes/no) [default=yes]?
Would you like to use an existing block device (yes/no) [default=no]?
Would you like LXD to be available over the network (yes/no) [default=no]?
Would you like stale cached images to be updated automatically (yes/no) [default=yes]?
Would you like to create a new network bridge (yes/no) [default=yes]?
What should the new bridge be called [default=lxdbr0]?
What IPv4 address should be used (CIDR subnet notation, “auto” or “none”) [default=auto]?
```

You should now be able to run `lxd.lxc list` without errors. It may warn you about running `lxd init`, but don't worry about that if your initialization succeeded.

### libertine ###

Now we're onto the easy part. `libertine` is only available from `edge` channels in the app store, but we're fairly close to having a version that we could push into more stable channels. For the latest and greatest libertine:

``` bash
$ sudo snap install --edge libertine
$ sudo snap connect libertine:libertined-client libertine:libertined
```

If we want libertine to work fully, we need to jump through a couple of hoops. For starters, dbus-activation is not fully functional at this time for snaps. Lucky for us, we can fake this by either running the d-bus service manually (`/snap/bin/libertined`), or by adding this file to `/usr/share/dbus-1/services/com.canonical.libertine.Service.service`

``` bash /usr/share/dbus-1/services/com.canonical.libertine.Service.service
[D-BUS Service]
Name=com.canonical.libertine.Service
Exec=/snap/bin/libertine.libertined --cache-output
```

Personally, I always create the file, which will allow libertined to start automatically on the session bus whenever a user calls it. Hopefully d-bus activation will be fixed sooner rather than later, but this works fine for now.

Another issue is that existing deb-based libertine binaries may conflict with the snap binaries. We can fix this by adjusting `PATH` in our `.bashrc` file:

``` bash $HOME/.bashrc
# ...
export PATH=/snap/bin:$PATH
```

This will give higher priority to snap binaries (which should be the default, IMO). One more thing to fix before running full-force is to add an environment variable to `/etc/environment` such that the correct libertine binary is picked up in Unity 8:

``` bash /etc/environment
# ...
UBUNTU_APP_LAUNCH_LIBERTINE_LAUNCH=/snap/bin/libertine-launch
```

OK! Now we're finally ready to start creating containers and installing packages:

``` bash
$ libertine-container-manager create -i my-container
# ... (this could take a few minutes)
$ libertine-container-manager install-package -i my-container -p xterm
# ... (and any other packages you may want)
```

If you want to launch your apps in Unity 7 (why not?):

``` bash
$ libertine-launch -i my-container xterm
# ... (lots of ourput, hopefully an open window!)
```

When running Unity 8, your apps should show up in the app drawer with all the other applications. This will all depend on libertined running, so make sure that it runs at startup!

I've been making a lot of improvements on the snap lately, especially as the ecosystem continues to mature. One day we plan for a much smoother experience, but this current setup will let us work out some of the kinks and find issues. If you want to switch back the deb-based libertine, you can just install it through `apt` and remove the change to `/etc/environment`.
