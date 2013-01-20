---
layout: post
title: "Pushing an Application to Heroku that Uses Ruby and Mongo"
date: 2013-01-20 11:55
comments: true
categories: pokephile ruby sinatra mongoid database heroku mongolab
---

_This is Part 3 in a multi-part series to detail the creation of a "simple" project combining [Ruby][ruby], [MongoDB][mongodb], [RSpec][rspec], [Sinatra][sinatra], and [Capybara][capybara] in preperation for a larger-scale side project set to begin January 2013. For more in this series, see the [Pokephile category][series-tag]. Part 4 of this series details moving from a development environment to a production environment using [Heroku][heroku]. The code for this side-project is located [on Github][pokephile], and the final product can be found [here][app]._

[ruby]: http://www.ruby-lang.org/
[mongodb]: http://www.mongodb.org/
[rspec]: http://rspec.info/
[sinatra]: http://www.sinatrarb.com/
[capybara]: https://github.com/jnicklas/capybara
[nokogiri]: http://nokogiri.org/
[pokephile]: https://github.com/larryprice/Pokephile
[series-tag]: /blog/categories/pokephile
[app]: http://pokephile.herokuapp.com
[heroku]: http://heroku.com/

[Heroku][heroku] is a hosting service for different types of web applications. The best thing about Heroku is it's free, you get a decent subdomain for your application, and there's no spam email. Go ahead and [sign up][heroku-signup] if you don't already have an account.

[heroku-signup]: http://api.heroku.com/signup

Now we need the Heroku Toolbelt. I'll illustrate for Ubuntu 12.10, but there's also [documentation for installing on any OS][installing].

[installing]: https://toolbelt.heroku.com/

``` bash
$ wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh
```

The toolbelt installs some Heroku-specific applications in addition to ensuring you have Foreman and [Git][git] on your system. Now it's time to tell the Heroku Toolbelt who we are.

[git]: http://git-scm.com/

``` bash Step 3 - Modified Excerpt from the Heroku Getting Started guide
$ heroku login
Enter your Heroku credentials.
Email: larry@example.com
Password: 
Could not find an existing public key.
Would you like to generate one? [Yn] 
Generating new SSH public key.
Uploading ssh public key /home/larry/.ssh/id_rsa.pub
```

In order for Heroku to figure out what ruby gems are needed to run your application, you need to specify a Gemfile. It's wise to specify a specific version of Ruby, and it's also a good idea to keep the gems' versions close to the state you developed with. For the [Pokephile][series-tag] application created in this series, this is my Gemfile:

``` ruby project/Gemfile
source :rubygems

ruby '1.9.3'

gem 'sinatra', '~>1.3.2'
gem 'haml', '~>3.1.6'
gem 'mongoid', '~>3.0.14'

group :development, :test do
	gem 'capybara', '~>2.0.1'
	gem 'rspec', '~>2.11.0'
	gem 'nokogiri', '~>1.5.5'
end
```

First I specify a source for my gems: 'rubygems' defaults to "http://rubygems.org" and hasn't failed me yet. Next I specify that I want to use Ruby 1.9.3, a necessity because Mongoid 3.x doesn't work correctly with 1.9.2. The versions of the first three gems were chosen by typing the following in the command line to determine which version I had installed on my machine:

``` bash
$ gem query | grep 'sinatra\|haml\|mongoid'
haml (3.1.7, 3.1.6)
mongoid (3.0.17, 3.0.15, 3.0.14)
sinatra (1.3.3, 1.3.2)
sinatra-contrib (1.3.2)
sinatra-reloader (1.0)
```

The '~>' operator tells Bundler to use greater-than-equal but stop before next highest version. So, for Sinatra '~>1.3.2' means that Bundler will accept anything greater-than-or-equal-to '1.3.2' and less than '1.4.0.' I tend to rely on the '~>' operator so I can be sure no APIs are changed in my gems.

The next block is a conditional checking in which environment the gems are being installed. This defaults to :development if none is specified. I put the gems used for testing in this block since they're not needed to run the application, but a developer/tester would need these to run the tests.

For this Gemfile to be meaningful, we need to use a program called [Bundler][bundler] to "bundle" the gems and their dependencies in a Gemfile.lock file.

[bundler]: http://gembundler.com/

``` bash
$ pwd
project/
$ sudo apt-get install bundler
...
$ bundle install
...
Your bundle is complete! Use `bundle show [gemname]` to see where a bundled gem is installed.
```

Running bundler will create a Gemfile.lock file and make sure your system has the specified gems. If some gems are missing or need new versions to be installed, bundler will ask the user for their password to get the required gems.

The next step for setting things up is to set up Git. If your application is already using Git, you only need to commit all files to verify that the Gemfile and Gemfile.lock make it into the repository.

``` bash
$ cd project/
$ git init
$ git add .
$ git commit -m "Initial commit."
```

Now we create a Heroku project and give it a meaningful name. If you can't think of a meaningful name, use 'heroku create' and Heroku will come up with something for you.

``` bash
$ heroku create meaningful-name
Creating meaningful-name... done, stack is cedar
http://meaningful-name.herokuapp.com/ | git@heroku.com:meaningful-name.git
Git remote heroku added
```

We're almost there. Because we included accessing Mongo databases in our application, we have to take care of that on the web. The easiest way to do that is using a Heroku Add-on. At this time, there are two major Heroku Add-ons for Mongo databases: [MongoLab][mongolab] and [MongoHQ][mongohq]. Both services have a starter service for $0/month, which is pretty awesome in my opinion. I flipped a coin and picked MongoLab for this application. Adding the add-on to our project:

[mongolab]: https://addons.heroku.com/mongolab
[mongohq]: https://addons.heroku.com/mongohq

``` bash
$ heroku addons:add mongolab:starter
```

If you haven't already, Heroku will ask you to "verify your account" before continuing. This means that you have to put in some credit card information. Note that you will not be charged, I guess Heroku just wants some indication that you might eventually pay for something. After you put in your credit card information, you may need to run the above command again.

Now that MongoLab is set up on the server-side, we need to tell Mongoid how to connect to that server. The following command will give you the environment variable needed to connect to the server:

``` bash
$ heroku config | grep MONGOLAB_URI
```

Now we update our mongoid.yml file to use that string:

``` yml project/mongoid.yml
development:
  sessions:
    default:
      database: dev
      hosts:
        - localhost
  options:

test:
  sessions:
    default:
      database: test
      hosts:
        - localhost

production:
  sessions:
    default:
      uri: <%= ENV['MONGOLAB_URI'] %>
      options:
        skip_version_check: true
        safe: true
```

Because of the way my application works, I want to prepopulate the database with some Pokemon.

``` bash
$ export MONGOLAB_URI=`heroku config | grep MONGOLAB_URI | cut -c 15-`
$ pwd
project/
$ cd tools/populate
$ irb
>> require './populater'
true
>> require 'mongoid'
true
>> Mongoid.load! '../../mongoid.yml', :production
{"sessions"=>{"default"=>{"uri"=>nil, "options"=>{"skip_version_check"=>true, "safe"=>true}}}}
>> Populater.new.add_pokemon 1000
nil
```

With the production database populated, we need to set an environment variable in our production application defining the environment.

``` bash
$ heroku config:add MONGOID_ENV=production
```

Now that we've made changes to the mongoid.yml file, we should commit again and push to Heroku.

``` bash
$ git commit -a -m "Updating mongoid.yml file for production"
...
$ git push heroku master
```

And that's it! Check your Heroku URL to make sure everything looks okay and call it a day, or make some upgrades as I did for my [personal version of this project][app].
