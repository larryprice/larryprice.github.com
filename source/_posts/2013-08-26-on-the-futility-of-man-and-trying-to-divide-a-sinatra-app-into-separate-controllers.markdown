---
layout: post
title: "On the Futility of Man and Trying to Divide a Sinatra App into Separate Controllers"
date: 2013-08-26 22:35
comments: true
categories: ruby sinatra web mvc retro
---

Oh, [Sinatra](http://www.sinatrarb.com/). You're oh-so-very dear to me. You made it so easy for me to write my [first](http://capitalpunishment.herokuapp.com) [web](http://pokephile.herokuapp.com) apps. All I had to do was write a couple routes and throw together a few HTML-like files and I had a web app. I used pattern matching to reduce the web-facing code for [Capital Punishment](https://github.com/larryprice/CapitalPunishment) from ~500 lines of code to <100 lines of code. You are perfect for writing small-time web applications.

But what about large web applications? What about a web app that has normal users and admin users, makes lots of database reads and writes (my previous apps only did reads from a user-facing perspective), and has to be able to show the history of everything, forever, to the authorized users who request it?

You see, Sinatra is kind of an [MVC framework](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), but not exactly. In Sinatra, you have Views (your HTML inter-mixed with Ruby code in your desired DSL) and you have Controllers (each of your routes). When a database is involved, you can use something like [ActiveRecord](https://github.com/bmizerany/sinatra-activerecord) or [Mongoid](http://mongoid.org/en/mongoid/index.html) or [DataMapper](http://datamapper.org/) and you have yourself Models.

So every route is kind of a Controller. Every. Route. In Capital Punishment, there were once 8 routes (there are now 7). In the project I've been working on recently, there are currently 56 routes. 56 routes in the language described above means I kind of have 56 controllers.

That's been pretty overwhelming, especially since the traditional way of creating routes in Sinatra is to shove them all in the same file. There are a few ways I could think of to address this. The way we chose six months ago (for better or worse) was found [on StackOverflow](http://stackoverflow.com/questions/5877000/what-is-a-controller-in-sinatra), and involves creating a bunch of different files where you shove all related routes. So you get this situation:

``` ruby app.rb
class App < Sinatra::Base
end

require_relative 'controllers/helpers'

Dir.glob("#{File.dirname(__FILE__)}/controllers/*.rb").each do |file|
  require file.chomp(File.extname(file))
end

class App
  get '/' do
    erb :home
  end
end
```

``` ruby controllers/reports_controller.rb
class App
  get '/reports/user_bills' do
    erb :user_bills_report
  end
  ...
end
```

And so on and so forth. This works fine for a while, but we've ended up with 13 "controller" files, many of which are not trivial. This also makes the App class quite large since its controllers handle most of the logic for the app. This also doesn't enforce any kind of URL-naming logic, so if a developer is working hard (s)he may create both `/reports/user_bills` and `user_info_reports` without realizing the inconsistency (s)he just injected into the system.

In hindsight, this method is not perfect. I think that the Rails method of individual controllers is significantly better for large apps. Some people have been using other methods for trying to make Sinatra more MVC, such as [sinatra-mvc](https://github.com/jorrizza/sinatra-mvc). To be frank, sinatra-mvc pretty much does the same thing we've done, but with more structure.

I think what I've learned is that you should use a tool for its intended purpose. Sinatra was written to quickly create web apps in Ruby with minimal effort. Once you have more than 10-15 routes, you should reconsider whether your app can still be called "minimal effort." Sinatra may fly you to the moon, but you're unlikely to see what spring is like on Jupiter or Mars.
