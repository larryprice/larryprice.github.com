---
layout: post
title: "Schemaless databases with Ruby and MongoDB"
date: 2013-01-05 16:54
comments: true
categories: ruby mongodb database rspec tdd nosql nokogiri pokephile
---

This is Part 1 in a multi-part series to detail the creation of a "simple" project combining [Ruby][ruby], [MongoDB][mongodb], [RSpec][rspec], [Sinatra][sinatra], and [Capybara][capybara] in preperation for a larger-scale side project set to begin January 2013. For more in this series, see the [Pokephile category][series-tag]. Part 1 details getting started with MongoDB and creating a collection using data scraped off the web using [Nokogiri][nokogiri]. The code for this side-project is located [on Github][pokephile].

[ruby]: http://www.ruby-lang.org/
[mongodb]: http://www.mongodb.org/
[rspec]: http://rspec.info/
[sinatra]: http://www.sinatrarb.com/
[capybara]: https://github.com/jnicklas/capybara
[nokogiri]: http://nokogiri.org/
[pokephile]: https://github.com/larryprice/Pokephile
[series-tag]: /blog/categories/pokephile

###A little background###

NoSQL is a database service used when working with a large amount of data that doesn't fit a relational model (read: [wikipedia][wiki-nosql]). It allows for mass storage without the overhead of SQL relations. There are many types of schemaless database services ([here's a list][wiki-tax]), but in particular I've been looking into what's called "Document Store."

[wiki-nosql]: http://en.wikipedia.org/wiki/Nosql
[wiki-tax]: http://en.wikipedia.org/wiki/Nosql#Taxonomy

Documents can be any number of key-value fields with a unique id. Document Store services usually encode data in a simple format such as XML, YAML, JSON, or BSON for storage purposes. MongoDB is a document store service which uses BSON to store documents. In Mongo, we connect to a specific database and then we can look through "collections," which are more-or-less equivalent to "tables" in relational databases.

###What about MongoDB and the Ruby driver?###

The first step is to get MongoDB working on your machine. Install MongoDB for your system -- on Ubuntu 12.10 I do this:

```
sudo apt-get install mongodb mongodb-dev mongodb-clients mongodb-server
```

Then we start up the daemon:

```
sudo service mongodb start
```

###What's the concept?###

The concept here is that we are going to have a database populated with [Pokemon][pokemon]. The user types a Pokemon's name into a search field and submits the form, which brings up an image of the Pokemon and some useful information.

[pokemon]: http://www.pokemon.com/

###Getting started###

Since I would like to focus on MongoDB, we can start by populating our database with Pokemon. If you're not familiar with Pokemon, there are lots of them (~650 at the date of this blog post). For my purposes, I may want to only add the first ~150 Pokemon, or I may want to add every Pokemon imaginable. I want it to be easy to add more if any new ones are added. So I'm going to start this project by creating a Populater, and we're going to use TDD to help us create it.

If you don't have RSpec installed, it's as easy as opening up a shell and:

```
$ sudo gem install rspec mongo
```

I'm going to put the Populater in a tools directory, and I'm going to put my spec files in a test/spec directory. The directory structure I want to use is as follows:

```
project
--tools
----populate
----test
------spec
```

In the 'tools/test/spec' directory, I create 'populater_spec.rb.' We'll write our first test:

``` ruby project/tools/test/spec/populater_spec.rb
describe Populater do
	describe "#new" do
		it "does not throw when creating instance" do
			expect {Populater.new}.to_not raise_error
		end
	end
end
```

The syntax for RSpec is mostly pseudo-English, so it's fairly straightforward to follow. The first 'describe' block says that we are describing the Populater class. The second 'describe' block says that we are describing the 'new' method of the 'Populater' class. The inner-most block is our test. We want to make sure that no exception is thrown when we create a new Populater. To run this test, open a terminal and type:

``` bash Running Rspec
$ pwd
~/project/tools/test
$ rspec populater_spec.rb
```

We get a big fat compile error, obviously due to the fact that there's no such thing as a 'Populater' class. So create the file 'populater.rb' in 'project/tools/populate' and create the class:

``` ruby project/tools/populate/populater.rb
class Populater
end
```

And include the 'Populater' class in our spec file:

``` ruby project/tools/test/spec/populater_spec.rb
require_relative '../../populate/populater'

describe Populater do
	describe "#new" do
		it "does not throw when creating instance" do
			expect {Populater.new}.to_not raise_error
		end
	end
end
```

Now run rspec. Hooray, we're passing all our tests! Let's add another test and some let's have RSpec do a little work before each test.

``` ruby project/tools/test/spec/populater_spec.rb
require_relative '../../populate/populater'

describe Populater do
	before:each do
		@populater = Populater.new
	end

	describe "#new" do
		it "does not throw when creating instance" do
			expect {Populater.new}.to_not raise_error
		end

		it "takes no params and returns a Populater instance" do
			@populater.should be_an_instance_of Populater
		end
	end
end
```

The 'before:each' syntax tells RSpec to perform this action before running each test. This way, we don't have to type out 'Populater.new' in each test. When we run RSpec, this test passes. Now let's actually do something meaningful in our new call. We want the Populater to empty all Pokemon from our database as it begins. In order to do this, we need to also tell the Populater what database to use, so we'll refactor slightly to pass in the name of our database to the Populater.

``` ruby project/tools/test/spec/populater_spec.rb
require_relative '../../populate/populater'
require 'mongo'

describe Populater do
	before:all do
		@col = Mongo::Connection.new.db('test')["pokemons"]
	end
	before:each do
		@populater = Populater.new('test')
	end

	describe "#new" do
		it "does not throw when creating instance" do
			expect {Populater.new('test')}.to_not raise_error
		end

		it "takes one param and returns a Populater instance" do
			@populater.should be_an_instance_of Populater
		end

		it "empties pokemon collection" do
			@col.insert({:test => "hi there"})
			@col.find.count.should_not eql 0
			Populater.new('test')
			@col.find.count.should eql 0
		end
	end
end
```

Similar to the 'before:each' syntax, the 'before:all' syntax runs the statement once. Here we want to get a handle to the 'pokemons' collection from our 'test' database. In our test, we run a 'find' with no arguments on the 'pokemons' collection to query everything in that collection. We also have an 'insert' statement where we insert an arbitrary document into our collection. You'll note later that this garbage document looks nothing like the Pokemon documents we insert, which is just another reason to love document-store databases. We run RSpec and we fail the test. Let's open up 'populater.rb' and fix this.

``` ruby project/tools/populate/populater.rb
require 'mongo'

class Populater
	def initialize(db_name)
		@col = Mongo::Connection.new.db(db_name)["pokemons"]
		@col.remove
	end
end
```

Test fixed. We connect to the same database and access the same collection and remove all the old data on intialize. So now we actually want to add Pokemon to the collection. We'll pick up a new 'describe' block for an 'add_pokemon' method. We'll then test that calling it with 0 adds no Pokemon to the collection.

``` ruby project/tools/test/spec/populater_spec.rb
...
describe "#add_pokemon" do
	it "adds 0 pokemon given 0" do
		@populater.add_pokemon 0
		@col.find.count.should eql 0
	end
end
...
```

When we run our tests, we get a NoMethodError and fail. We create a trivial fix in populater.rb

``` ruby project/tools/populate/populater.rb
class Populater
	...

	def add_pokemon(num)
	end
end
```

And we pass the test, having added 0 Pokemon to our database. Let's do it with 1 now.

``` ruby project/tools/test/spec/populater_spec.rb
...
describe "#add_pokemon" do
	...
	it "adds 1 pokemon given 1" do
		@populater.add_pokemon 1
		@col.find.count.should eql 1
	end
end
...
```

We fail. Another trivial fix:

``` ruby project/tools/populate/populater.rb
class Populater
	...

	def add_pokemon(num)
		(0...num).each do |x|
			@col.insert({number: x})
		end
	end
end
```

We pass again. We'll also pass when checking for multiple Pokemon:

``` ruby project/tools/test/spec/populater_spec.rb
...
describe "#add_pokemon" do
	...
	it "adds 2 pokemon given 2" do
		@populater.add_pokemon 2
		@col.find.count.should eql 2
	end
end
...
```

But we're missing substance. There's only garbage being shoved in our database. Our TDD methodology breaks down slightly here because we want our database to have dynamic information scraped from a website, and I don't want to hard code any data nor do I want to scrape the same website in my tests and my implementation. So we're going to do a little bit of behind-the-scenes stuff and test that the fields we want are simply not nil. I want each Pokemon to have a number, name, an array of types, and a link to an image:

``` ruby project/tools/test/spec/populater_spec.rb
...
describe "#add_pokemon" do
	...
	it "adds pokemon with a number" do
		@populater.add_pokemon 1
		@col.find.count.should eql 1
		@col.find.first['number'].should_not be_nil
	end
	it "adds pokemon with a name" do
		@populater.add_pokemon 1
		@col.find.count.should eql 1
		@col.find.first['name'].should_not be_nil
	end
	it "adds pokemon with array of types" do
		@populater.add_pokemon 1
		@col.find.count.should eql 1
		types = @col.find.first['types']
		types.should_not be_nil
		types.should be_an_instance_of Array
		types.should have_at_least(1).items
		types.should have_at_most(2).items
	end
	it "adds pokemon with image link" do
		@populater.add_pokemon 1
		@col.find.count.should eql 1
		image = @col.find.first['image']
		image.should_not be_nil
		image.should_not be_empty
	end
end
...
```

There are many websites where you can get this kind of data for each Pokemon, but I chose [the Pokemon Wiki][pokewiki] for its consistency. In the initializer of the Populater, I open up the URL using Nokogiri so I can access the sweet, creamy data contained within. In my add_pokemon method, I extract this data I want based on the way the table is set up on the website. To continue, we need to install the Nokogiri gem:

[pokewiki]: http://pokemon.wikia.com/wiki/List_of_Pok%C3%A9mon

```
sudo gem install nokogiri
```

And now we add the logic to add_pokemon:

``` ruby project/tools/populate/populater.rb
require 'mongo'
require 'nokogiri'
require 'open-uri'

class Populater
	def initialize(db_name)
		@col = Mongo::Connection.new.db(db_name)["pokemons"]
		@col.remove
		@data = Nokogiri::HTML(open("http://pokemon.wikia.com/wiki/List_of_Pok%C3%A9mon"))
	end

	def add_pokemon(num_to_add)
		@data.xpath("//table[@class='wikitable sortable']/tr").each do |row|
			break if num_to_add <= 0
			dex_num = row.at_xpath('td/text()').to_s.strip
			next if dex_num.nil? || dex_num.empty?
			dex_name = row.at_xpath('td[2]/a/text()').to_s.strip

			unless dex_num == "000"
				type_1 = row.at_xpath('td[4]/a/span/text()').to_s.strip
				type_2 = row.at_xpath('td[5]/a/span/text()').to_s.strip || row.at_xpath('td[5]/text()').to_s.strip
				image_link = "http://img.pokemondb.net/artwork/#{dex_name.downcase}.jpg"
			else
				type_1 = row.at_xpath('td[4]/text()').to_s.strip
				type_2 = row.at_xpath('td[5]/text()').to_s.strip
				image_link = "images/missingo.png"
			end

			types = Array.new
			types << type_1 unless type_1.nil? || type_1.empty?
			types << type_2 unless type_2.nil? || type_2.empty?

			@col.insert({:number => dex_num, :name => dex_name, :types => types, :image => image_link})

			num_to_add -= 1
		end
	end
end
```

I'll admit The add_pokemon method is now quite a bit more daunting to interpret. Here's the breakdown of what's going on: Nokogiri finds us the table tag with class of 'wikitable sortable' and we iterate over that. There are two breaking conditions of our loop: we hit the max number of Pokemon as given, or we can't find anymore Pokemon in the table. So we check that we haven't hit our max. Then we find the Pokemon's number in the table after we manually parse the HTML. In the case of this table, the first row is all garbage, so we continue to the next row if we are on the first row.  We then grab the name from the table, which is luckily always in the same place. The branch is for the special case of Pokemon #000 (Missingo), which is set up slightly differently in the table for some reason. We create an empty array and shove our types in it, but we have to be careful because not all Pokemon have two types. We then create a document in the braces and insert it into the collection. The final step is to decrement the loop counter.

Tests pass. We now have a working Populater! Now we can either write a script or open up the irb and populate as necessary and we know that the Populater is functional:

``` ruby Populating Databases
$ irb
>> Dir.pwd
=> "project/tools/populate"
>> require 'mongo'
=> true
>> col = Mongo::Connection.new.db('dev')["pokemons"]
=> ...
>> col.find.count
=> 0
>> require './populater'
=> true
>> Populater.new('dev').add_pokemon 152
=> nil
>> col.find.count
152
>>
```

If you want to further familiarize yourself with the MongoDB Ruby driver, you should check out the MongoDB Koans. Unfortunately, the original [MongoDB Koans][koans] have not been updated in a while, and so my more recent installations of Ruby and the MongoDB driver didn't work. I found a set of [updated koans][updated-koans] which worked with my install of Ruby 1.9.3. However, the updated version also had a couple of annoying issues with deprecations, so I created [my own fork][my-koans] on GitHub with the fixes.

[mongodb-setup]: http://net.tutsplus.com/tutorials/databases/getting-started-with-mongodb/
[koans]: https://github.com/tredfern/MongoDB_Koans
[updated-koans]: https://github.com/edgecase/ruby_koans
[my-koans]: https://github.com/larryprice/MongoDB_Koans
