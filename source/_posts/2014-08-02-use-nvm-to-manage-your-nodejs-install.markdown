---
layout: post
title: "Use NVM to manage your NodeJS install"
date: 2014-08-02 20:00:57 -0400
comments: true
categories: nodejs nvm
---

I've been trying to get into [NodeJS](http://nodejs.org/), and so my immediate thought is that I want to be able to install different versions for different projects, a la [rvm](https://rvm.io) for [ruby](https://ruby-lang.org). Fortunately, this already exists.

[NVM](https://github.com/creationix/nvm) gives me the equivalent functionality of rvm but for NodeJS, making all my dreams come true. Here's how I installed it on my Linux boxes:

``` bash
$ curl https://raw.githubusercontent.com/creationix/nvm/v0.12.2/install.sh | bash
```

*Note: This URL will get you nvm version 0.12.2. This link may not be valid in the future, where you come from. Check out the [github repo](https://github.com/creationix/nvm) for any newer versions. If you're brave, trusting, or just really naïve, you can even change `v0.12.2` above to `master` to get the bleeding edge install.*

The above line of code will download the files, install nvm in your home directory, and update your profile to include nvm's current Node version in your path. NVM autocomplete isn't in place by default, but we can enable it by adding the following to the end of our .bash_profile:

``` bash
[[ -r $NVM_DIR/bash_completion ]] && . $NVM_DIR/bash_completion
```

Now it's time to actually install us some Node! You can use `nvm ls-remote` to list the versions of Node currently available for download. At the time of this writing, the most recent version is **v0.11.13**. Installing is easy (and quick):

``` bash
$ nvm install 0.11.13
...
$ nvm use 0.11.13
```

Since this is the only Node on my system, I'd like to set it as the default.

``` bash
$ nvm alias default 0.11.13
```

What if I'm `cd`ing out of control and I don't know what Node version I need in my current directory? Create a file called `.nvmrc` in the directory containing the version number you want nvm to use and then type `node use` ENTER; Now you're using the version of Node you meant to. This also prevents people from using the wrong versions of Node to try to run your code, which would of course be a catastrophe.

PLUS nvm installs the right version of npm whenever you install Node, so there's no need to worry about dealing with your base npm not working with different versions of Node.

Thank [FOSS](https://en.wikipedia.org/wiki/Free_and_Open_Source_Software) for nvm. Maybe one day soon I'll do useful things in Node an tell you about them.
