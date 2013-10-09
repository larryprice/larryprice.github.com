---
layout: post
title: "RVM Quick Start"
date: 2013-10-08 22:10
comments: true
categories: ruby rvm
---

When I was working on a few different projects at once, I started running into issues where my Ruby gem versions would start to mismatch. How did I fix that issue? Naively. I adjusted the versions as necessary so my gems were always the same version. That was... pretty dumb. To make up for my past ignorance, I've been exploring ways to manage my Ruby versions and my gems intelligently. Enter [RVM](https://rvm.io/).

RVM is a simple tool to solve just problems. And it works pretty well. I hit some kinks along the way, but my installation pleases me well enough. Getting RVM is not for the faint of heart. Maybe there are better ways to do it, but the website says to execute the following:

``` bash /home/lrp/my_project
> curl -L https://get.rvm.io | bash -s stable
```

This command uses curl to fetch the data that lives at `https://get.rvm.io`, which happens to be a big bash script. It executes the bash script with the args `-s stable`. 'Stable' means the latest stable release of RVM. I could just as easily give it 'dev' and get the latest developer release instead, but I really don't want that. So that command does a lot of stuff and may give you some further instructions to run before you can continue. Use that big head of yours and follow the instructions. You may also need to restart a terminal to get RVM to be recognized as a command. Since I was using `gnome-terminal`, I had to follow these [instructions](https://rvm.io/integration/gnome-terminal).

Now I want RVM to know about some Ruby versions. For a full list of possible Ruby versions to install, run `rvm list known`. I just want 1.9.3. I found that I needed to do this even though I had 1.9.3 installed on my system previously.

``` bash /home/lrp/my_project
> rvm install ruby-1.9.3
```

In Ubuntu 13.04, this command installs ruby-1.9.3 in `~/.rvm/rubies/ruby-1.9.3-p448/bin/ruby`. I've found that this also sets my default Ruby to the RVM version of Ruby, which I don't want. To verify and undo this, I executed the following commands outside of my project directory.

``` bash /home/lrp
> which ruby
/home/lrp/.rvm/rubies/ruby-1.9.3-p448/bin/ruby
> echo "D'oh"
D'oh
> rvm use system
Now using system ruby.
> which ruby
/usr/bin/ruby
>
```

Now that that's settled, I want to tell RVM to use the local version of Ruby for my project and to install any gems in a special location.

``` bash /home/lrp/my_project
> rvm 1.9.3@my-project --create --ruby-version
```

This creates a gemset and ruby-version file (`.ruby-gemset` and `.ruby-version` files) using the Ruby installation 1.9.3 created above. I specifiy to use `--ruby-version` instead of `--rvmrc` because RVM told me that I should. After some research, the `.ruby-version` file is used by several other tools, so this will keep my potential number of config files low. Now I check that all my Ruby versions are okay.


``` bash /home/lrp/my_project
> which ruby
/home/lrp/.rvm/rubies/ruby-1.9.3-p448/bin/ruby
> cd ..
> which ruby
/usr/bin/ruby
>
```

When in my project directory, all the gems I install will be installed to my specified gemset, which means they are no longer cluttering my global gemspace, even when I install them using Bundler. It also means that I can use ruby-1.9.3 for this project and 2.0 for another project with minimal mental overhead. This makes me a happy developer.

I only glazed over the installation process and documentation for RVM. Go to [the web site](https://rvm.io/) for more.
