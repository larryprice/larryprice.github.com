---
layout: post
title: "Javascript testing with Capybara and Cucumber"
date: 2014-08-05 06:40:40 -0400
comments: true
categories: ruby capybara cucumber sinatra testing javascript
---

In the past, I had written off testing the Javascript in my [Sinatra](http://www.sinatrarb.com/) apps as being not worth the pain of setting up. That was pretty naïve of me, as setting up web drivers in [Capybara](https://github.com/jnicklas/capybara) is actually pretty easy.

For this post, I assume you already have a capybara-cucumber project set up. If you need help setting up your first tests, consider checking out [some of my other blog posts](/blog/categories/capybara/) on the subject.

####Selenium

Selenium is the default Javascript driver for capybara. To install either run `gem install selenium-webdriver` or toss `selenium-webdriver` into your `Gemfile` and run `bundle install`.

If you want to run all of your tests with Javascript enabled, you can change the default driver in your `env.rb` file. For example:

``` ruby env.rb
ENV['RACK_ENV'] = 'test'

require_relative '../../../web'

require 'selenium-webdriver'
require 'capybara/cucumber'
require 'rspec'

Capybara.app = Ollert
Capybara.default_wait_time = 10

# run all tests using Javascript
Capybara.default_driver = :selenium

World do
  Ollert.new
  Mongoid.purge!
end
```

Using Selenium means that your tests will be running using Firefox. Unfortunately, this makes them much, much slower than when you were running the tests using rspec. What I recommend is to limit yourself to only use the Javascript driver when you need to. To accomplish this, we change our `env.rb` file as such:

``` ruby env.rb
ENV['RACK_ENV'] = 'test'

require_relative '../../../web'

require 'selenium-webdriver'
require 'capybara/cucumber'
require 'rspec'

Capybara.app = Ollert
Capybara.default_wait_time = 10

# use the following web driver to run tests
Capybara.javascript_driver = :selenium

World do
  Ollert.new
  Mongoid.purge!
end
```

And add the `@javascript` tag to our Cucumber feature files. I've written [about using tags in the past](/blog/2013/04/15/tags-in-c-plus-plus-cucumber-tests/), and they are incredibly useful. For example:

``` cucumber DoStuff.feature
Feature: Do Stuff

Scenario: Nothing happens with Javascript disbaled
  Given I visit the home page
  When I click "Where are they?!"
  Then I should not see "I'm Batman."

@javascript
Scenario: Correct text is displayed with Javascript enabled
  Given I visit the home page
  When I click "Where are they?!"
  Then I should see "I'm Batman."
```

Assuming there is some Javascript activated by clicking "Where are they?!" that displays the text "I'm Batman.", both of the above scenarios will pass. This is because none of the Javascript will run in the first scenario, so the text will not be displayed. In the second scenario, Capybara knows to use the webdriver we set up previously when it sees the `@javascript` tag.

####Poltergeist

[Poltergeist](https://github.com/teampoltergeist/poltergeist) is the first "headless" web driver I tried. Usage is mostly identical to usage for Selenium, so I'll focus on installation here.

Poltergeist uses [PhantomJS](http://phantomjs.org/), so we need to start by downloading the binary ([32-bit](https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-1.9.7-linux-i686.tar.bz2) or [64-bit](https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-1.9.7-linux-x86_64.tar.bz2)) and putting it in our path. On my Linux machine, I extracted the contents of the download and copied `bin/phantomjs` to my `/usr/local/bin` directory, which I already have in my path. You can also copy it directly to `/usr/bin` if you like.

On the ruby side, we do that same old song and dance: either do `gem install poltergeist` or add `poltergeist` to your `Gemfile` and run `bundle install`. Edit your `env.rb`:

``` cucumber env.rb
ENV['RACK_ENV'] = 'test'

require_relative '../../../web'

require 'capybara/poltergeist'
require 'capybara/cucumber'
require 'rspec'

Capybara.app = Ollert
Capybara.default_wait_time = 10

# use the following web driver to run tests
Capybara.javascript_driver = :poltergeist

World do
  Ollert.new
  Mongoid.purge!
end
```

Now your Javascript tests should be running using the Poltergeist webdriver. Since Poltergeist is truly headless, your tests will run much faster than they did while using Selenium, but you won't be able to see what's going on while your tests run. There are some slight syntactic differences between the way Poltergeist and Selenium handles separate windows, but other than that they are extremely similar.

####Webkit

[Capybara-webkit](https://github.com/thoughtbot/capybara-webkit) is where I eventually landed for running my own tests, after having issues accessing other windows with Poltergeist. Capybara-webkit is also headless and relies on `QtWebKit` to render pages. So, for starters, you're going to have to install `qtwebkit`. This has a varied degree of difficulty depending on which operating system you're using, but I didn't have too many problems in Ubuntu once I figured out which library I needed. For help, check [the guide](https://github.com/thoughtbot/capybara-webkit/wiki/Installing-Qt-and-compiling-capybara-webkit). On my machine:

``` bash
$ sudo apt-get install libqtwebkit-dev
```
Once more: either do `gem install capybara-webkit` or add `capybara-webkit` to your `Gemfile` and run `bundle install`. Edit your `env.rb`:

``` cucumber env.rb
ENV['RACK_ENV'] = 'test'

require_relative '../../../web'

require 'capybara/cucumber'
require 'capybara/webkit'
require 'rspec'

Capybara.app = Ollert
Capybara.default_wait_time = 10

# use the following web driver to run tests
Capybara.javascript_driver = :webkit

World do
  Ollert.new
  Mongoid.purge!
end
```

Again, you won't be able to see your tests run, but they should be pretty snappy. I was able to use capybara-webkit to tackle some window issues I was having, but (as of this writing) capybara-webkit has not caught up with more modern capybara window-switching syntax. Other than that, the syntax is identical to the other drivers I've discussed for common cases. If you're running capybara-webkit on a CI server, see [this post about using Xvfb](http://blog.55minutes.com/2013/09/running-capybara-webkit-specs-with-jenkins-ci/).
