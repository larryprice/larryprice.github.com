---
layout: post
title: "Convincing rvm to let you use ruby 2.1.0"
date: 2014-02-22 16:46:00 -0500
comments: true
categories: ruby rvm linux
---

[Ruby 2.1.0 went stable](//ruby-lang.org/en/news/2013/12/25/ruby-2-1-0-is-released/) a few months ago, and [Ruby 1.9.3 support](//ruby-lang.org/en/news/2014/01/10/ruby-1-9-3-will-end-on-2015/) will end in a just over a year.

You know what that means: Warplanes in the sky falling to the ground, dogs and cats getting along like old pals, and people wandering aimlessly through the streets trying to remember the last time they saw a green build.

Believe it or not, all of these things can be prevented. Once upon a time, I wrote about [configuring and using rvm](/blog/2013/10/08/rvm-quick-start/) to control individual ruby environments for each of your projects. If it's been a while since you installed your copy of [rvm](//rvm.io/), you might have some trouble installing and using newer version of ruby. Lucky for us, those clever rvm developers made it easy to get around this.

In my case, I want to upgrade a project to use `ruby 2.1.0`. The first time I tried to run `rvm install ruby-2.1.0`, I ended up installing `ruby-2.1.0-preview1`. I realized that I had installed rvm on this machine around October 2013, and `ruby 2.1.0` was released in December 2013, so rvm had no idea that `ruby 2.1.0` was stable. Updating rvm (in the root of the project directory):

``` bash /home/lrp/Projects/2014/projNeedingRuby210
$ rvm get stable
```

There will be some amount of text on the screen if your system needs to be updated. Note that you must be connected to the internet if you want rvm to update. Now we do the install of our brand new ruby:

``` bash /home/lrp/Projects/2014/projNeedingRuby210
$ rvm install ruby-2.1.0
```

Again, text on the screen from fetching of data from the internet. But hopefully you see something that tells you the operation was successful. You can also verify which rubies you have installed using the `list` command:

``` bash /home/lrp/Projects/2014/projNeedingRuby210
$ rvm list rubies
   ruby-1.9.3-p448 [ x86_64 ]
=> ruby-2.0.0-p247 [ x86_64 ]
 * ruby-2.1.0 [ x86_64 ]

# => - current
# =* - current && default
#  * - default
```

Now we tell our current project to use `ruby 2.1.0`:

``` bash /home/lrp/Projects/2014/projNeedingRuby210
$ rm .ruby-version .ruby-gemset
$ rvm 2.1.0@projNeedingRuby210 --create --ruby-version
ruby-2.1.0 - #gemset created /home/lrp/.rvm/gems/ruby-2.1.0@projNeedingRuby210
ruby-2.1.0 - #generating projNeedingRuby210 wrappers.
$ rvm gemset copy 2.0.0-p247@projNeedingRuby210 2.1.0@projNeedingRuby210
Copying gemset from 2.0.0-p247@projNeedingRuby210 to 2.1.0@projNeedingRuby210
Generating gemset wrappers ruby-2.1.0@projNeedingRuby210.
Making gemset 2.1.0@projNeedingRuby210 pristine.
$ which ruby
/home/lrp/.rvm/rubies/ruby-2.1.0/bin/ruby
```

Alright! Crisis averted. If you're using [bundler](//bundler.io/) with this project, be sure to change your ruby version (usually located near the top of the `Gemfile`).

What about setting up a new project using `ruby 2.1.0`? Easy! Switch to the project directory and:

``` bash /home/lrp/Projects/2014/newRubyProject
$ rvm 2.1.0@newRubyProject --create --ruby-version
```

Oh, rvm, you make life _too_ easy sometimes.
