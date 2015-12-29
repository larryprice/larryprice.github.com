---
layout: post
title: "Javascript: The Good Parts"
date: 2015-12-28 20:17:33 -0500
comments: true
categories: write-ups javascript web
---

### The Gist ###

[Javascript: The Good Parts](http://amzn.to/1OVjj3Z) by Douglas Crockford is a short guide to the best parts of the javascript language. Javascript is notorious for being so flexible and feature-rich that many traditional programmers start writing code and completely forget how to program and building a startup based on an idea they got from the [Bored Elon Musk](https://twitter.com/boredelonmusk) mock-Twitter account only to wake up a year later cold and alone living in the slums of San Francisco. This book is full of example code with thoughtful explanations to help the average developer write better javascript and leave the web a nicer place.

### Takeaways ###

I read this book because I thought the title was too snarky for its own good. Javascript is flexible and capable of doing nearly anything you need but, like any tool, can easily be misused by newcomers. Since javascript is the de facto language of the web, many developers who use javascript are novices or don't think they need to know the language constructs because "it's not a real language." Unfortunately, this results in a lot of spaghetti code and insecure javascript being processed by our web browsers.

I studied javascript this past fall to try to hone my skills, and I found that many of the little tricks I picked up through independent study had already been collected in this book. Anyone doing javascript should have a passing knowledge of the contents of this book before they are allowed to push to production.

Now I'll get off my soapbox and point out a few of my favorite points made in this book.

Javascripts most powerful structures are `Object` and `Array`. An `Array` is an `Object` with properties of type `Number` cast to integers in sequential order. This means that the `Array` object `[0, 1, "mario", "luigi"]` is represented as an `Object` as `{"0": 0, "1": 1, "2": "mario", "3": "luigi"}`.

`Object.hasOwnProperty` tells you whether the given property name is on the object's prototype or has been defined on the current object. This is very useful when using a `for in` loop to filter properties on an object.

Although I've frequently used the module pattern with a self-executing function to create private scope, I hadn't connected the dots that the reason this magic works is because of closures. Here's an example:

``` javascript
var MyModule = (function() {
  var x = 0;
  return {
    reset: function() {
      x = 0;
    },
    do: function() {
      console.log(x++);
    }
  }
}());

MyModule.reset();
MyModule.do();    // output: 0
MyModule.do();    // output: 1
MyModule.do();    // output: 2
MyModule.reset();
MyModule.do();    // output: 0
```

`MyModule` contains the private variable `x`, which cannot be accessed by anyone who has a reference to this function. However, the internal functions `reset` and `do` do have access to the private variable. Because of this, we know no external code can manipulate our internal value and we can guarantee some amount of consistency in our code given there was no tampering of the source. Of course, closures are useful for far more than just scoping variables.

Although there are many ways to mimic traditional inheritance in javascript with `new` and defining the `constructor` method on a prototype, it usually makes more sense to use _differential inheritance_. Given an object named `bird`, we'll make a new object named `parrot` using `var parrot = Object.create(bird);` and start setting new properties on `parrot` This type of inheritance uses the basics of the language to our advantage and adheres nicely to the native prototypal constructs.

### Action Items ###

* __Differential inheritance.__ I've always tried to force traditional inheritance into my javascript architecture, and it's always felt wrong. The next time I need inheritance, I will try to use DI to utilize javascript's prototype system better.
* __Write more javascript.__ It's been a while since I got to write javascript professionally, so I'd like to ensure I maintain my knowledge base over the coming months by writing more javascript on the side. As web programming is where I want to be, hopefully I can get back into building for the web before the end of the year.
