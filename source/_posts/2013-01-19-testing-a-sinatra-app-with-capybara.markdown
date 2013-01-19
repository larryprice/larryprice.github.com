---
layout: post
title: "Testing a Sinatra App with Capybara"
date: 2013-01-19 12:20
comments: true
categories: pokephile ruby sinatra capybara cucumber haml mongoid database twitter-bootstrap
---

_This is Part 3 in a multi-part series to detail the creation of a "simple" project combining [Ruby][ruby], [MongoDB][mongodb], [RSpec][rspec], [Sinatra][sinatra], and [Capybara][capybara] in preperation for a larger-scale side project set to begin January 2013. For more in this series, see the [Pokephile category][series-tag]. Part 3 of this series describes using Capybara to test a Sinatra application. The code for this side-project is located [on Github][pokephile], and the final product can be found [here][app]._

[ruby]: http://www.ruby-lang.org/
[mongodb]: http://www.mongodb.org/
[rspec]: http://rspec.info/
[sinatra]: http://www.sinatrarb.com/
[capybara]: https://github.com/jnicklas/capybara
[nokogiri]: http://nokogiri.org/
[pokephile]: https://github.com/larryprice/Pokephile
[series-tag]: /blog/categories/pokephile
[app]: http://pokephile.herokuapp.com

Now that [the database is populated][populate-post] with data and I've switched over to [a simpler interface with Mongo][mongoid-post], I can actually start creating a UI. For simplicity's sake, I like to use Sinatra on small projects. Sinatra makes it easy to create a web application with minimal effort. With an emphasis on testing this [series][series-tag], I want to be sure to throughly test my UI and any application integration. [Cucumber][cukes] is a brilliant DSL which allows a programmer to describe in plain English how an application should be behaving. The [Capybara gem][capybara] is a Rack app that simulates running your application and performing basic user tasks, such as clicking a button, following a link, or, on a lower level, looking at your HTML source. Install Capybara like so:

``` bash
sudo gem install capybara
```

[populate-post]: /blog/2013/01/05/schemaless-databases-with-ruby-and-mongodb/
[mongoid-post]: /blog/2013/01/05/moving-from-the-mongodb-ruby-driver-to-mongoid/
[cukes]: http://cukes.info/

Here's what I want my application to do:

* User is on home page. 
* User enters name of Pokemon and presses 'Search'. 
* User is redirected to search page. 
* User can see some information about Pokemon. 
* User can repeat the search process.

Error scenario:

* User enters garbage data.
* User is redirected to search page.
* User sees error message.
* User can repeat search process.

So I'll begin by writing my features. Cucumber syntax is meant to be readable by non-technical persons, so the "code" may look a bit odd. All Cucumber feature files are written something like this:

``` Cucumber
Feature: Viewer visits the Home Page
  In order to read the page
  As a viewer
  I want to see the home page of my app

Scenario: View the home page
	Given I am on the home page
	Then I should see "Zowee, mama!" on the page
```

The "Feature" lines are not used in testing; they are simply to give some relevance to the file's feature and to attempt to prevent scope creep in the file. The "Scenario" lines are what's important. The first line is the test name, the "Given" line is the pre-condition, and the final line is what should be observed.

In reality, I have only one main feature: "Search." One can argue that I also have a feature of "seeing" my home page and my search page, but those are both trivial cases, so for the purpose of this blog post, I'll skip such tests. Both my success and error case revolves around searching, and there's not really much else to do in the app. I have to create a directory for the cukes, and that directory is called "features." I create such a directory in my "project/tools/test" directory and create a new file called "search.feature." Now I'll write the feature:

``` Cucumber project/tools/test/features/search.feature
Feature: Viewer vists the page
	In order to search the page
	As a visitor
	I want to search for Pokemon.

Scenario: Find correct Pokemon from home page
	Given I am on the home page
	When I type "Bulbasaur" in the search bar
	And I click "Search"
	Then I should be on the "search" page
	And I should see "#001 - Bulbasaur"
	And I should see an image with url "http://img.pokemondb.net/artwork/bulbasaur.jpg"
	And I should see "Types: Grass, Poison"

Scenario: Show error text from home page
	Given I am on the home page
	When I type "Johnny Bravo" in the search bar
	And I click "Search"
	Then I should be on the "search" page
	And I should see "Lol! Could not find a Pokemon named 'Johnny Bravo.' Try something else!"

Scenario: Find correct Pokemon from search page
	Given I am on the search page
	When I type "Bulbasaur" in the search bar
	And I click "Search"
	Then I should see "#001 - Bulbasaur"
	And I should see an image with url "http://img.pokemondb.net/artwork/bulbasaur.jpg"
	And I should see "Types: Grass, Poison"

Scenario: Show error text from search page
	Given I am on the search page
	When I type "Johnny Bravo" in the search bar
	And I click "Search"
	Then I should be on the "search" page
	And I should see "Lol! Could not find a Pokemon named 'Johnny Bravo.' Try something else!"
```

Now I've overlooked a lot of tests that one would normally write while doing this, such as verifying that the search bar and search buttons exist and are enabled, but I'd like to keep it simple for now and just stick to testing my search feature. What do these tests do? The first scenario starts on the home page, enters data in the search box, presses the search button, and then verifies that all expected Pokemon data is visible. When writing cukes, I can append statements with an "And" statement as seen above. Run Cucumber:

``` sh project/tools/test
$ cucumber
```

Cucumber doesn't exactly give us errors, but it also doesn't give us success. Fortunately, what it did give us was sample code for all of the steps we need to write. So, let's perform some copy/paste magic and create a steps file:

``` sh project/tools/test
$ mkdir -p features/step_definitions
$ touch features/step_definitions/search_steps.rb
```

``` ruby project/tools/test/features/step_definitions/search_steps.rb
Given /^I am on the home page$/ do
  pending # express the regexp above with the code you wish you had
end

When /^I type "(.*?)" in the search bar$/ do |arg1|
  pending # express the regexp above with the code you wish you had
end

When /^I click "(.*?)"$/ do |arg1|
  pending # express the regexp above with the code you wish you had
end

Then /^I should be on the "(.*?)" page$/ do |arg1|
  pending # express the regexp above with the code you wish you had
end

Then /^I should see "(.*?)"$/ do |arg1|
  pending # express the regexp above with the code you wish you had
end

Then /^I should see an image with url "(.*?)"$/ do |arg1|
  pending # express the regexp above with the code you wish you had
end

Given /^I am on the search page$/ do
  pending # express the regexp above with the code you wish you had
end
```

If I futilely run "cucumber" again now, my tests still don't pass because I haven't actually implemented my steps. This is where Capybara comes in. I found that a [Capybara cheat sheet][cheat] is quite helpful while writing out my steps. The syntax I'm going to use is similar to RSpec, except that it includes some Capybara methods. The first two test steps I want to deal with are the "Given" steps.

[cheat]: https://gist.github.com/428105

``` ruby project/tools/test/features/step_definitions/search_steps.rb
Given /^I am on the home page$/ do
  visit '/'
end

Given /^I am on the search page$/ do
  visit '/'
  click_button "Search"
end
```

All these statements are just Regular Expressions, as indicated by the /^$/. The regex acts as a sort of method name that Cucumber finds to run the steps. Given I am on the home page is trivial: just 'visit' the index. Given I am on the search page will first require me to click the "search" button. This is valid because my spec above says this is how to get to the search page. Now can do the 'When' statements.

``` ruby project/tools/test/features/step_definitions/search_steps.rb
When /^I type "(.*?)" in the search bar$/ do |arg1|
  fill_in "pokemon-input", :with => arg1
end

When /^I click "(.*?)"$/ do |arg1|
  click_button arg1
end
```

When I click just needs to click a button/link/whatever on the screen. The "(.*?)" is a regular expression that will match anything in quotes and assign it to the variable 'arg1.' So I can give any button description and Capybara will try to click a button with the given content. When I type in the search bar takes the regex arg1 and uses the "fill_in" method to fill in a text input with id "pokemon-input." The rest of the steps are all about what should be observed after performing the Given/When steps.

``` ruby project/tools/test/features/step_definitions/search_steps.rb
Then /^I should be on the "(.*?)" page$/ do |arg1|
  current_path.should == "/#{arg1}"
end

Then /^I should see "(.*?)"$/ do |arg1|
  page.should have_content(arg1)
end

Then /^I should see an image with url "(.*?)"$/ do |arg1|
  find(:xpath, "//img[@src='#{arg1}']").should_not be_nil
end
```

Now I have all of the necessary steps defined! So, I'll run Cucumber and... Actual errors! None of my four tests made it past the Given step, so I see the output '(4 failed, 19 skipped).' The only way to fix these errors is to finally start writing a web application. So I'll move back out to the root of my project directory and create a file for my application called 'app.rb' and give it the most basic information to run. And, if you haven't already, install Sinatra.

``` bash
$ sudo gem install sinatra
```

``` ruby project/app.rb
require 'sinatra'
require 'haml'

class Application < Sinatra::Base
	get '/' do
		haml :index
	end
end
```

The Application class inherits from the Sinatra::Base class. This allows me to define a 'get' operation to perform actions and load a web page. 'get \'\' do' signifies the first page a user sees when they go to my web application, commonly known as a home or index page. I plan to use [HAML][haml] to create my page, so I make a call to haml followed by the name of my HAML document as a symbol. We need to define an 'index.haml' page and stick it in a directory called 'views' for Sinatra to find it.

[haml]: http://haml.info

``` bash
$ sudo gem install haml
```


``` haml project/views/index.haml
!!!
%html
	%head
		%title Pokemon App
	%body
		LOL HAI.
```

Tough work. If you're not familiar with HAML, it's a markup language that is "compiled" into an HTML page. The main difference between HAML and HTML is that HAML parses white space to figure out where closing tags should be placed. So now I want to run my application. I want to run it using 'rackup,' so I'd like to define a 'config.ru' file in the root of my project directory to do all the work for me.

``` ruby project/config.ru
require './app'

run Application.new
```

Now we run 'rackup' from the root of my project directory and see the fruits of my labor. Open up a web browser and enter 'localhost:9292' in the address bar. You should see a very simple web page with the content of "LOL HAI" and a title of "Pokemon App." If you view the source, you'll see the HTML the HAML was compiled into. Just beautiful, isn't it? Now if I switch back to my test directory and run Cucumber, what happens? The same result. That's because I need to tell Capybara what to load before trying to run the tests. I do this by defining an "env.rb" file in a "support" directory of the features directory.

``` ruby project/tools/test/features/support/env.rb
require 'capybara'
require 'capybara/cucumber'

require_relative "../../../../app"

Capybara.app = Application
```

All I do is require my "app.rb" file which is seemingly _forever_ away and then set the Capybara.app variable to my Application class. Now I run Cucumber and... '(4 failed, 17 skipped, 2 passed)' Two steps passed! Yippee! Now if only the rest passed as well. Looking at my 'search.feature' file, I can see that the first 'When' step is about typing into the search bar. So my first design decision is what kind of search bar I want. I've opted for the fun way out: using the [Twitter Bootstrap's][boots] [typeahead][typeahead]. The typeahead has functionality to give suggestions while the user types, and the best news is this is already coded for us. Adding the code for my search bar and a search button:

[boots]: http://twitter.github.com/bootstrap/
[typeahead]: http://twitter.github.com/bootstrap/javascript.html#typeahead

``` haml project/views/index.haml
!!!
%html
	%head
		%title Pokemon App
		%link(rel="stylesheet" href="http://twitter.github.com/bootstrap/assets/css/bootstrap.css")
	%body
		%script{:type => "text/javascript", :src  => "http://code.jquery.com/jquery.min.js"}
		%script{:type => "text/javascript", :src  => "http://twitter.github.com/bootstrap/assets/js/bootstrap-typeahead.js"}
		#search{:style => "position: absolute; width: 100%; text-align: center; top: 10%;"}
			%form{:action => "search", :method => "POST", :id => "pokemon-search"}
				%input{:type => "text", :class => "input-large", :id => "pokemon-input", :name => "pokemon", "data-provide"=>"typeahead", "data-items"=>"10", "autocomplete"=>"off", :autofocus => "", :placeholder => "Find a Pokémon...", "data-source" => Pokemon.only(:name).map {|x| x.name}}
				%button{:type=>"submit", :class => "btn btn-small", :style => "margin-bottom: 10px; font-weight: bold;"}
					Search
```

In the \<head\> tag, I include a link to the Bootstrap stylesheet. In the \<body\> tag, I include a link to the JQuery and Bootstrap Typeahead JavaScript files remotely so I don't have to keep track of them. I then add a \<div\> tag called "search" and center it on the page. Inside the div tag I create a form whose action sends a POST signal to the "search" action. Inside the form is first the typeahead, then a small submit button. The important parameters in the typeahead are "data-items" and "data-source;" "data-items" tells the JavaScript function how many items to suggest at a time, and "data-source" is an array of data for the JavaScript to search. Notice that my "data-source" uses the Pokemon class [created previously][populate-post], so I need to be able to set up a [Mongoid connection][mongoid-post] to access that data. I'll make this connection in my "config.ru" file:

``` ruby project/config.ru
require './app'
require 'mongoid'

class Application
	configure do
		Mongoid.load! 'mongoid.yml'
	end
end

run Application.new
```

I have chosen to extend the Application class in my "config.ru" file to prevent interference with my test setups later. Taking a look at the application would be a good idea, but if I run "rackup" now Mongoid will complain about environment setup. By default, "Mongoid.load!" will try to load the "development" settings, so I need to include a "development" setup in my "mongoid.yml." For now, it's going to be identical to my "test" environment setup except for the database name:

``` yml project/mongoid.yml
test:
  sessions:
    default:
      database: test
      hosts:
        - localhost

development:
  sessions:
    default:
      database: dev
      hosts:
        - localhost

```

And ensuring some Pokemon are in the "dev" database:

``` bash Populating the dev database
$ irb
>> Dir.pwd
=> "project/tools/populate"
>> require 'mongoid'
=> true
>> Mongoid.load! '../../mongoid.yml'
=> {"sessions"=>{"default"=>{"database"=>"dev", "hosts"=>["localhost"]}}}
>> require './populater'
=> true
>> Populater.new.add_pokemon 152
=> nil
>>
```

And requiring the Pokemon model in app.rb:

``` ruby project/app.rb
require 'sinatra'
require 'haml'

require_relative 'pokemon'

class Application < Sinatra::Base
	get '/' do
		haml :index
	end
end
```

Finally, run 'rackup' from the root of the 'project' directory and load up the web application at 'localhost:9292.' There's now a typeahead and a search button in the top center of the page, and typing in the 'Search' bar shows up to 10 suggestion Pokemon. Now I'll return to my Cucumber tests. I need to add a line in the 'env.rb' file to set up the Mongoid environment and ensure there are Pokemon in the collection.

``` ruby project/tools/test/features/support/env.rb
require 'capybara'
require 'capybara/cucumber'
require 'mongoid'

require_relative '../../../populate/populater'
require_relative "../../../../app"

Mongoid.load! '../../mongoid.yml', :test

Populater.new.add_pokemon(10)

Capybara.app = Application
```

Now I can run Cucumber and see some jovial results: '(4 failed, 9 skipped, 10 passed).' I now have more steps passing than failing! The root cause of the failures is that there currently is no "search" page; let's fix that:

``` ruby project/app.rb
...
class Application < Sinatra::Base
	...
	post '/search' do
		haml :search
	end
end
```

I want my search page to have a search bar just like my index page. If I want them to be identical, I want to only have to change that code once. When writing HAML, I can create a 'layout.haml' file to act as a base page for my application and move all the text from 'index.haml.' I'll add a '=yield' statement where I want the information from 'index.haml' and 'search.haml' to be placed.

``` haml project/views/layout.haml
!!!
%html
	%head
		%title Pokemon App
		%link(rel="stylesheet" href="http://twitter.github.com/bootstrap/assets/css/bootstrap.css")
	%body
		%script{:type => "text/javascript", :src  => "http://code.jquery.com/jquery.min.js"}
		%script{:type => "text/javascript", :src  => "http://twitter.github.com/bootstrap/assets/js/bootstrap-typeahead.js"}
		#search{:style => "position: absolute; width: 100%; text-align: center; top: 10%;"}
			%form{:action => "search", :method => "POST", :id => "pokemon-search"}
				%input{:type => "text", :class => "input-large", :id => "pokemon-input", :name => "pokemon", "data-provide"=>"typeahead", "data-items"=>"10", "autocomplete"=>"off", :autofocus => "", :placeholder => "Find a Pokémon...", "data-source" => Pokemon.only(:name).map {|x| x.name}}
				%button{:type=>"submit", :class => "btn btn-small", :style => "margin-bottom: 10px; font-weight: bold;"}
					Search
		=yield
```

``` haml project/views/index.haml
#search-text{:style => "position: absolute; width: 100%; text-align: center; top: 2%; font-weight: bold;"}
	Begin typing to search for your Pokemon!
```

At this point, I'll create a 'search.haml' file in the 'views' directory, but leave it empty. Running Cucumber now, I get '(4 failed, 4 skipped, 15 passed).' Pretty close! All I fail now is actually seeing the desired information on the page. First I want to get access to the Pokemon searched for: I can do that by parsing the params passed to us in app.rb:

``` ruby project/app.rb
...

class Application < Sinatra::Base
	...
	post '/search' do
		@pokemon = Pokemon.where(name: params[:pokemon]).first
		haml :search
	end
end
```

So now on my search page:

``` haml project/views/search.haml
#search-text{:style => "position: absolute; width: 100%; text-align: center; top: 2%; font-weight: bold;"}
	Search for another Pokemon.
#search-results{:style => "position: absolute; width: 100%; text-align: center; top: 20%; font-weight: bold;"}
	%img{:src => @pokemon.image, :height => "250px"}
	%br
	= "##{@pokemon.number} - #{@pokemon.name}"
	%br
	= "Types: #{@pokemon.types.first}#{@pokemon.types.count < 2 ? ' ' : ', ' + @pokemon.types.last}"
```

I reference the class variable "@pokemon" and access its data. To output Ruby-formatted strings, I use an "=" sign. Running Cucumber, I now see '(2 failed, 21 passed).' 2 of my 4 tests are passing! A trivial amount of investigation reveals that I didn't deal with the situation where the user types in garbage data. That can mostly be done in the HAML file, but I also want a way to get the bad text the user gave me so they can see what was wrong.

``` haml project/views/search.haml
#search-text{:style => "position: absolute; width: 100%; text-align: center; top: 2%; font-weight: bold;"}
	Search for another Pokemon.
#search-results{:style => "position: absolute; width: 100%; text-align: center; top: 20%; font-weight: bold;"}
	- unless @pokemon.nil?
		%img{:src => @pokemon.image, :height => "250px"}
		%br
		= "##{@pokemon.number} - #{@pokemon.name}"
		%br
		= "Types: #{@pokemon.types.first}#{@pokemon.types.count < 2 ? ' ' : ', ' + @pokemon.types.last}"
	- else
		= "Lol! Could not find a Pokemon named '#{@name}.' Try something else!"
```

``` ruby project/app.rb
...
class Application < Sinatra::Base
	...
	post '/search' do
		@name = params[:pokemon]
		@pokemon = Pokemon.where(name: @name).first
		haml :search
	end
end
```

Ruby code with no output is preceded with a '-' and because HAML is space-sensitive, there's no need to include 'end' statements. Now I run Cucumber and... 4 scenarios/23 steps passed! I have a functional web application! Of course, the page itself is somewhat bland, some of the styles could be put in a stylesheet and reused, and the only tests I've written are for super-high-level functionality. Those are all problems someone with infinite time would deal with, so I'll just leave the page is is for now.

The next blog post [in this series][series-tag] will be about deploying this application to the web using Heroku.
