---
layout: post
title: "Fetching Random Mongoose Objects the Simple Way"
date: 2014-09-15 20:11:13 -0400
comments: true
categories: nodejs mongoose mongodb database javascript
---

As I venture through the land of [NodeJS](http://nodejs.org/), I've found the wonder and magic of [NPM](http://npmjs.org/), a package management tool for Javscript similar to [ruby](https://www.ruby-lang.org/)'s gems. Although there are nearly 100,000 packages on the main npmjs site (94,553 at time-of-writing), it seems there are still holes to be filled.

Recently, while working on a _top secret side-project_, I wanted to grab a random object from a [MongoDB](https://www.mongodb.org/) collection. I used the highly-extensible [mongoose](http://mongoosejs.com/) to set up my models and just needed to find a package somewhere with the desired functionality. I found such a package called [mongoose-random](https://github.com/matomesc/mongoose-random), but, unfortunately, I was never able to get this plugin to work correctly. The plugin in question also needed to insert new columns on your tables, which I didn't really want. So I decided to create a new package.

[mongoose-simple-random](https://www.npmjs.org/package/mongoose-simple-random) is an incredibly easy way to include a random accessor on your mongoose models. All that's required is adding the plugin to the schema before compiling the model:

``` javascript test.js
var random = require('mongoose-simple-random');

var s = new Schema({
  message: String
});
s.plugin(random);

Test = mongoose.model('Test', s);
```

Now I can ask the model for a single random element of the `Test` model with a single call to `findOneRandom`:

``` javascript example.js
var Test = require('./test');

Test.findOneRandom(function(err, element) {
  if (err) console.log(err);
  else console.log(element);
});
```
