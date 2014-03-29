---
layout: post
title: "Using sqlite on Heroku"
date: 2014-03-29 11:16:06 -0400
comments: true
categories: sql heroku ollert ruby sinatra sequel
---

Or rather, "Not Using sqlite on Heroku."

[Heroku](//heroku.com) does not support [sqlite](//sqlite.org). That doesn't mean we have to stop using sqlite in development, but it does mean we need to put in some workarounds to support our deployment environment. The rest of this article will use [ruby](//ruby-lang.org) and [Sinatra](//sinatrarb.com).

Assuming you have a heroku app deployed and you have sqlite already working locally, this only takes a few steps. First we need to add a SQL database to our heroku app. From the project directory, we'll add the [heroku-postgresql](//addons.heroku.com/heroku-postgresql) addon to our app.

``` bash
$ heroku addons:add heroku-postgresql:dev
```

The `dev` piece of this command tells heroku we want the small, free database. This database supports up to 10,000 rows and has a 99.5% uptime. Best of all: it's free. Other options have you pay $9/mo for 10,000,000 rows or $50+ for Unlimited usage. I recommend you start small.

Hopefully you got some success statements after adding heroku-postgresql. They should have included some new environment variables, which are links to your new Postgres database. Record these; we'll use them a little later.

Now we need to set up the back-end to be able to access a Postgres database when necessary. Hopefully you're using a decent abstraction library in your app that can access any SQL database. For ruby, I find [Sequel](//www.sequel.rubyforge.org/) to be sufficient.

In our Gemfile, we've probably already included the sqlite gem for use in our local environment. We can go ahead and move that into a `development` block, and we need to add the `pg` gem to either `production` or the global block.

``` ruby Gemfile
source "https://rubygems.org"

ruby '2.1.0'

gem 'bundler'
gem 'rake'
gem 'sinatra'
gem 'haml'
gem 'sequel'

group :production do
  gem 'pg'
end

group :development do
  gem 'sqlite3'
end
```

Heroku sets `ENV['RACK_ENV']` to "production" for us, which means that the pg gem should get picked up the next time we deploy. Now we need to tell our app which database to use in which situation.

One of the easiest places to make this decision is in Sinatra's `configure` block. I keep my local db in an environment variable called `LOCAL_DATABASE_URL`. This is where you use the environment variable heroku set for you when you set up your Postgres database; mine was called `HEROKU_POSTGRESQL_MAROON_URL`.

``` ruby web.rb
class App < Sinatra::Base
  configure :production do
    Sequel.connect ENV['HEROKU_POSTGRESQL_MAROON_URL']
  end

  configure :development do
    Sequel.connect ENV['LOCAL_DATABASE_URL']
  end
end
```

This works because the default environment is "development." Test locally, and then we can deploy.

``` bash
$ git push heroku master
```

And enjoy.
