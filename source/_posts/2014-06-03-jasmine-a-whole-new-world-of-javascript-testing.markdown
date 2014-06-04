---
layout: post
title: "Jasmine - A Whole New World of Javascript Testing"
date: 2014-06-04 6:19:43 -0400
comments: true
categories: javascript testing
---

[Jasmine](https://jasmine.github.io/): a headless Javascript testing library written entirely in Javascript. With similarities to [rspec](http://rspec.info), I've quickly grown attached to this framework and have been looking for opportunties to discuss it. [Version 2.0](https://jasmine.github.io/2.0/introduction.html) was recently released, so I'll be focusing on the standalone 2.0 concepts. To get started, download and uncompress [the standalone distribution](https://github.com/pivotal/jasmine/tree/master/dist).

The uncompressed directory structure will have three subdirectories: `spec`, `src`, and `lib`. `lib` contains all the Jasmine source code. `src` contains some sample Javascript class that is tested by test files contained in `spec`. Outside of the subdirectories is the special file `SpecRunner.html`. This file is how we will run our tests.

Let's start a new pizza place.

We'll need Pizza. A Pizza will need several things: size, style, toppings, and price. We'll have a few styles available, but also allow our guests to request additional toppings. We'll also set the price based on the size and number of toppings. Create the files `src/pizza.js` and `spec/PizzaSpec.js` and add them to `SpecRunner.html`.

We'll start by being able to get the styles from Pizza.

``` js spec/PizzaSpec.js
describe("Pizza", function() {
  var pizza;

  beforeEach(function() {
    pizza = new Pizza();
  });

  it("should give a choice of styles", function() {
    expect(pizza.getStyles()).toContain("meat lovers");
    expect(pizza.getStyles()).toContain("veg head");
    expect(pizza.getStyles()).toContain("supreme");
  });
});
```

The syntax is just lovely: We use `describe` to set visual context, `beforeEach` to perform a task before each spec, and `it` to encapsulate a test. The results of running `SpecRunner.html` in my browser:

```
Pizza should give a choice of styles
  TypeError: pizza.getStyles is not a function in file:///home/lrp/docs/jasmine/spec/PizzaSpec.js (line 9)
```

Fixing it:

``` js src/pizza.js
function Pizza() {
  this.getStyles = function() {
    return ["meat lovers", "veg head", "supreme"];
  }
}
```

And the results:

```
Pizza
    should give a choice of styles
```

Let's set the toppings:

``` js spec/PizzaSpec.js
describe("Pizza", function() {
  // ...

  describe("toppings", function() {
    it("should have no toppings when no style and no extras given", function() {
      pizza.initialize();
      expect(pizza.getToppings().length).toBe(0);
    });

    it("should have only extras when no style and extras given", function() {
      var extras = ["pineapple", "edamame", "cheeseburger"]
      pizza.initialize(null, null, extras);
      
      expect(pizza.getToppings().length).toBe(extras.length);
      for (var i = 0; i < extras.length; i++) {
        expect(pizza.getToppings()).toContain(extras[i]);
      }
    });

    it("should have special toppings when given style and extras", function() {
      var extras = ["pineapple", "edamame", "cheeseburger"];
      pizza.initialize(null, "veg head", extras);
      
      expect(pizza.getToppings().length).toBe(7);
    });

    it("should have special toppings when given style", function() {
      var extras = ["pineapple", "edamame", "cheeseburger"];
      pizza.initialize(null, "veg head");
      
      expect(pizza.getToppings().length).toBe(4);
    });
  });
});
```

For these tests, I nested a describe block to give better context to what I'm testing. Fixing the tests:

``` js src/pizza.js
function Pizza() {
  // ...

  var size, toppings;

  function findToppings(style, extras) {
    toppings = extras ? extras : [];

    switch (style) {
      case ("meat lovers"):
        toppings.push("ham", "pepperoni", "bacon", "sausage");
        break;
      case ("veg head"):
        toppings.push("onion", "tomato", "pepper", "olive");
        break;
      case ("supreme"):
        toppings.push("pepperoni", "onion", "sausage", "olive");
        break;
    }
  }

  this.getToppings = function() {
    return toppings;
  };

  this.initialize = function(pizzaSize, style, extras) {
    size = pizzaSize;
    findToppings(style, extras);
  };
}
```

And finally, I'll deal with the cost. I'll come out of scope of the nested `describe` and nest another `describe`.

``` js spec/PizzaSpec.js
describe("Pizza", function() {
  // ...

  describe("cost", function() {
    it("is detemined by size and number of toppings", function() {
      pizza.initialize(10, "supreme");
      expect(pizza.getToppings().length).toBe(4);
      expect(pizza.getCost()).toBe(7.00);
    });

    it("is detemined by size and number of toppings including extras", function() {
      pizza.initialize(18, "meat lovers", ["gyros", "panchetta"]);
      expect(pizza.getToppings().length).toBe(6);
      expect(pizza.getCost()).toBe(12.00);
    });
  });
});
```

To fix this test, I'll use my handy-dandy pizza-cost forumla:

``` js src/pizza.js
function Pizza() {
 // ...

  this.getCost = function() {
    return size/2 + toppings.length * .5;
  }

  // ...
}
```

This is great and all, but a bit simple. What if we wanted to make an ajax call? Fortunately, I can fit that into this example using [Online Pizza](http://onlinepizza.se/api/), the pizza API. Unfortuantely, the API is kind of garbage, but that doesn't make this example any more meaningless. You can [download jasmine-ajax on Github](https://github.com/pivotal/jasmine-ajax/raw/master/lib/mock-ajax.js), and stick it in your `spec/` directory and add it to `SpecRunner.html`. At this point I need to include [jquery]() as well.

In order to intercept ajax calls, I'll `install` the ajax mocker in the `beforeEach` and uninstall it in an `afterEach`. Then I write my test, which verifies that the ajax call occurred and returns a response.

``` js spec/PizzaSpec.js
beforeEach(function() {
  jasmine.Ajax.install();

  pizza = new Pizza();
});

afterEach(function() {
  jasmine.Ajax.uninstall();
});

describe("sendOrder", function() {
  it("returns false for bad pizza", function() {
    pizza.sendOrder();

    expect(jasmine.Ajax.requests.mostRecent().url).toBe("http://onlinepizza.se/api/rest?order.send");

    jasmine.Ajax.requests.mostRecent().response({
      status: "500",
      contentType: "text/plain",
      responseText: "Invalid pizza"
    });

    expect(pizza.orderSent()).toBe(false);
  });

  it("returns true for good pizza", function() {
    pizza.sendOrder();

    expect(jasmine.Ajax.requests.mostRecent().url).toBe("http://onlinepizza.se/api/rest?order.send");

    jasmine.Ajax.requests.mostRecent().response({
      status: "200",
      contentType: "text/plain",
      responseText: "OK"
    });

    expect(pizza.orderSent()).toBe(true);
  });
});
```

To get this to work, I add some logic to the `Pizza` class to set some state based on what the ajax call returns.

``` js src/pizza.js
var orderSuccess;

this.sendOrder = function() {
  orderSuccess = null;

  $.ajax({
    type: "POST",
    url: "http://onlinepizza.se/api/rest?order.send",
    success: function() {
      orderSuccess = true;
    },
    error: function() {
      orderSuccess = false;
    }
  });
}

this.orderSent = function() {
  return orderSuccess;
}
```

Ajax calls tested. By installing Jasmine's ajax mock, all of the ajax calls were intercepted and were not sent to the server at Online Pizza. Any ajax calls that may have been fired by the `Pizza` class but were not addressed in the spec are ignored. The final test results look something like this:

```
Pizza
    sendOrder
        returns false for bad pizza
        returns true for good pizza
    styles
        should give a choice of styles
    toppings
        should have no toppings when no style and no extras given
        should have only extras when no style and extras given
        should have special toppings when given style and extras
        should have special toppings when given style
    cost
        is detemined by size and number of toppings
        is detemined by size and number of toppings including extras
```

Full sample code [available on Github](https://github.com/larryprice/jasmine-pizza). There's a lot of other interesting things Jasmine can do that I'm still learning about. If applicable, I'll try to create a blog post for advanced Jasmine usage in the future.