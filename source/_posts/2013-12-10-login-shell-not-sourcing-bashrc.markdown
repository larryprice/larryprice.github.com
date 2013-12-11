---
layout: post
title: "Login shell not sourcing .bashrc: A brief lesson in dot-files"
date: 2013-12-10 19:45
comments: true
categories: linux rvm
---

I have a confession: I've been avoiding using [rvm](https://rvm.io/) for the past few weeks for stupid reasons.

When using `rvm` with `gnome-terminal`, I have to tell `gnome-terminal` to run as a login shell so that `/etc/profile` is sourced. The login shell is then supposed to source `$HOME/profile`, which is then supposed to source `$HOME/.bashrc`. Installing `rvm` results in me editing my `.bash_profile` to add the following line:

``` bash .bash_profile
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*
```

All my `rvm` environments seem to work, so this is a success, right?

Well, I type the good ol' `ls` command and I notice that all my colors are missing. I notice that the terminal title no longer reads the present working directory, but instead greets me with a disinterested "Terminal." What happened?

Eventually I realized that this was a problem caused by not sourcing my `.bashrc`. Where did I go wrong?

The login shell sources `$HOME/.profile` UNLESS `$HOME/.bash_profile` exists, in which case it only sources the latter. So what are the contents of my `$HOME/.profile`?

``` bash .profile
# ~/.profile: executed by the command interpreter for login shells.

# [cut for brevity]

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
      . "$HOME/.bashrc"
    fi
fi

# [cut for brevity]
```

Aha! The sneaky dot-file actually sources `$HOME/.bashrc`, and my shiny new `$HOME/.bash_profile` doesn't. I fixed this by sourcing `$HOME/.profile` in `$HOME/.bash_profile`.

``` bash .bash_profile
[ -f "$HOME/.profile" ] && source "$HOME/.profile"
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" # Load RVM into a shell session *as a function*
```

For the memory-impaired: `.bash_profile` sources `.profile` sources `.bashrc`.

Re-open `gnome-terminal` to find my colors are fixed, my title is correct, and `rvm` plays the right notes. The world is at peace once again, and I don't have to avoid using `rvm`.
