---
layout: post
title: "Using Foreman to Create an Upstart Service"
date: 2013-08-31 17:05
comments: true
categories: web ruby sinatra foreman upstart
---

I just finished my first attempt at deploying a web app to run automatically in the background on a friend's server. Pretty easy, really. The first thing I did was install [foreman](https://github.com/ddollar/foreman). Assuming you have ruby and rubygems installed:

``` bash
$ sudo gem install foreman
```

Next I needed to give foreman the commands to start my app. I created a file in the root of my project directory called 'Procfile' and gave it the steps I would run to start my app manually. For the sake of simplicity, let's say I run my app pretty barebones:

``` bash Procfile
web: bundle exec rackup
```

Now when I run `foreman start`, foreman will use [Bundler](http://bundler.io/) to execute rackup with the correct gems in my Gemfile. Now exporting upstart config files is pretty easy.

``` bash
$ sudo foreman export upstart --app=MyApp --user=root /etc/init
```

That command creates the .conf files needed for upstart to control the service called 'MyApp' as the user 'root.' It puts all the .conf files in `/etc/init` (which is where Ubuntu puts such things) and will create a default log directory in `/var/log/MyApp`. Now I can control my service by running `service MyApp start`, `service MyApp stop`, `service MyApp restart`, and `service MyApp status`. Hooray for me.

But I need to run my app in two ways: in dev mode on a local port with my dev database, and I need to run it in production mode using port 80 and the production database. I've also heard that using [webrick](https://en.wikipedia.org/wiki/WEBrick) (the default server installed with rackup) is great for develpment, but I should be using something else for my production server. So I made some config files for foreman:

``` bash development.env
RACK_ENV=development
PORT=9292
SERVER=rackup
```

``` bash production.env
RACK_ENV=production
PORT=80
SERVER=unicorn
```

And I change my Procfile to:

``` bash Procfile
web: bundle exec $SERVER -p $PORT -E $RACK_ENV
```

Ridiculously configured. Now when I run `foreman start`, it will error out. I need to specify my environment file:

``` bash
$ foreman start -e production.env
```

Now foreman will use Bundler to startup the server specified in `$SERVER`, run the app on port `$PORT` (-p), and will pass through the environment listed as `$RACK_ENV` to my application (-E), allowing my app to do whatever configuration it does given the current environment. Power to the people.

[I found that this guy](http://michaelvanrooijen.com/articles/2011/06/08-managing-and-monitoring-your-ruby-application-with-foreman-and-upstart/) does a lot more complicated stuff with Foreman, if you need more.
