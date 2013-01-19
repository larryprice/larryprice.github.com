---
layout: post
title: "Moving from the MongoDB Ruby Driver to Mongoid"
date: 2013-01-05 20:04
comments: true
categories: ruby mongodb mongoid database rspec nosql pokephile
---

This is Part 2 in a multi-part series to detail the creation of a "simple" project combining [Ruby][ruby], [MongoDB][mongodb], [RSpec][rspec], [Sinatra][sinatra], and [Capybara][capybara] in preperation for a larger-scale side project set to begin January 2013. For more in this series, see the [Pokephile category][series-tag]. Part 2 details refactoring code using the MongoDB Ruby driver to use Mongoid. The code for this side-project is located [on Github][pokephile].

[ruby]: http://www.ruby-lang.org/
[mongodb]: http://www.mongodb.org/
[rspec]: http://rspec.info/
[sinatra]: http://www.sinatrarb.com/
[capybara]: https://github.com/jnicklas/capybara
[pokephile]: https://github.com/larryprice/Pokephile
[series-tag]: /blog/categories/pokephile

###What I've Done

In a [previous post][prev], I described creating a class that would populate a database with data scraped from the internet. I used the MongoDB Ruby driver to accomplish this. However, using the driver can be laborious and there are simpler ways. In this post, I'm going to refactor the Populater class to use Mongoid.

[prev]: /blog/2013/01/05/schemaless-databases-with-ruby-and-mongodb/

###Mongoid###

[Mongoid][mongoid] (pronounced mann-goyd) is an Object-Document Wrapper for Ruby. Using mongoid abstracts some of the database operations that must be performed when using the MongoDB Ruby driver. It comes in handy when using models in an MVC application. To install the Mongoid gem:

[mongoid]: http://http://mongoid.org/en/mongoid/index.html

```
sudo gem install mongoid
```

###Refactoring###

In populater.rb, we only inserted one structure of document into our "pokemons" collection. That makes this a great opportunity to use Mongoid. We remember that there were four fields in our document: number (string), name (string), image link (string), and types (array). Knowing this, we can create a model for this data:

``` ruby project/pokemon.rb
require 'mongoid'

class Pokemon
	include Mongoid::Document

	field :number, type: String
	field :name, type: String
	field :types, type: Array
	field :image, type: String
end
```

And that's it for our model. Although we specified the types in this case, it's not necessary if we want a looser definition of our model. Here's how we change our implementation file:

``` ruby project/tools/populate/populater.rb
#require 'mongo' #deleted
require_relative '../../pokemon' #added
require 'nokogiri'
require 'open-uri'

class Populater
	#def initialize(db_name) #removed
  def initialize #added
      #@col = Mongo::Connection.new.db(db_name)["pokemons"] #deleted
      #@col.remove #deleted
      Pokemon.delete_all #added
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

        #@col.insert({:number => dex_num, :name => dex_name, :types => types, :image => image_link}) #deleted
        Pokemon.create {:number => dex_num, :name => dex_name, :types => types, :image => image_link} #added

        num_to_add -= 1
    end
  end
end
```

That one's easy. We deleted four lines and added 3. However, now you can see that the Populater does not have to deal with connecting to the database, it only has to know what model it wants to modify. So we've removed some complexity from this file by no longer requiring the database name on initialization. However, that means that someone else has to be in charge of setting up the initial connection. In the overlying project, we want that someone else to be a controller. In our tests, we want that someone else to be our test file. So let's do it. We're going to start by adding a config section in our before:all block.

``` ruby project/tools/test/spec/populater_spec.rb
require_relative '../../populate/populater'
#require 'mongo' #removed
require 'mongoid' #added
require_relative '../../../pokemon'

describe Populater do
	before:all do
		Mongoid.configure do |config| #added
			config.connect_to 'test' #added
		end # added
		#@col = Mongo::Connection.new.db('test')["pokemons"] # removed
	end
	...
end
```

In doing this, we've set up any of our document models to use the 'test' database. Now we go through each test and replace the Mongo Ruby Driver syntax with Mongoid syntax, which is similar to Ruby's Array syntax.

``` ruby project/tools/test/spec/populater_spec.rb
...

describe Populater do
	...
	before:each do
		#@populater = Populater.new('test') #removed
		@populater = Populater.new #added
	end

	describe "#new" do
		it "does not throw when creating instance" do
			#expect {Populater.new('test')}.to_not raise_error #removed
			expect {Populater.new}.to_not raise_error #added
		end

		it "takes one param and returns a Populater instance" do
			@populater.should be_an_instance_of Populater
		end

		it "empties pokemon collection" do
			#@col.insert({:test => "hi there"}) #removed
			#@col.find.count.should_not eql 0 #removed
			#Populater.new('test') #removed
			#@col.find.count.should eql 0 #removed
			Pokemon.create #added
			Pokemon.count.should eql 1 #added
			Populater.new #added
			Pokemon.count.should eql 0 #added
		end
	end
	...
end
```

The 'new' tests are straightforward. We remove the usage of an input parameter to the Populater initializer. The only significant change we make is to the "empties pokemon collection" test. Here we replace the Mongo Ruby Driver syntax of inserting into a collection with Mongoid syntax of creating a Pokemon document. The 'create' method inserts a document into the collection with the given values, or defaults if none are given. We also see that we can remove the "find" syntax completely and just use a "count" method on the document type.

``` ruby project/tools/test/spec/populater_spec.rb
...
describe Populater do
	...
	describe "#add_pokemon" do
		it "adds 0 pokemon given 0" do
			@populater.add_pokemon 0
			#@col.find.count.should eql 0 #removed
			Pokemon.count.should eql 0 #added
		end
		...
		it "adds pokemon with a number" do
			@populater.add_pokemon 1
			#@col.find.count.should eql 1 #removed
			#@col.find.first['number'].should_not be_nil #removed
			Pokemon.count.should eql 1 #added
			Pokemon.first['number'].should_not be_nil #added
  	end
	  ...
	end
end
```

The tests for adding 0, 1, and 2 documents to the collection are all very similar. The only change is to replace the Mongo Ruby Driver "find.count" syntax with the Mongoid "count." The "adds pokemon with a ____" tests all undergo the same changes. I replace the ".find.first" statement with a simple ".first" to get the same meaning. So our Populater has been converted to use Mongoid instead of the Mongo Ruby Driver. Bully for us.

There's one more change that would be nice to make before we hang up our hats. Configuring Mongoid using the .config syntax is okay, but it would be a lot nicer to keep all of our configuration in a file. We can create such a file called "mongoid.yml" and put some configuration information in it:

``` yml project/mongoid.yml
test:
  sessions:
    default:
      database: test
      hosts:
        - localhost
```

This syntax is valid in Mongoid 3.x. This is a very simply configuration for our test environment. Now we can go back into our test file and change the 'before:all' block:

``` ruby project/tools/test/spec/populater_spec.rb
...
describe Populater do
	before:all do
		#Mongoid.configure do |config| #removed
		#	config.connect_to 'test' #removed
		#end # removed
		Mongoid.load! '../../../mongoid.yml', 'test' #added
	end
	...
end
```

The second parameter can be a string or a symbol. Now there's only one file to modify the environment configurations, and we're better off for it.
