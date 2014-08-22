---
layout: post
title: "Hotfixing a Bug in Trello's client.js"
date: 2014-08-22 06:08:21 -0400
comments: true
categories: trello ollert javascript
---

I'm a huge fan of [Trello](https://trello.com), and I recently created an app to analyze Trello data called [Ollert](https://ollertapp.com). Ollert makes heavy use of the [Trello API](http://trello.com/docs). I've written previously about [using Trello's client.js to connect to the Trello API](/blog/2014/03/18/connecting-to-the-trello-api/).

During some ad-hoc testing of opening/closing the Trello authorization popup, I found that clicking "Deny" on the popup, reopening it, and then clicking "Connect" resulted in me not being able to connect. I spent some time looking into workarounds and I even contacted Trello support. After studying [client.coffee](https://trello.com/1/client.coffee), I eventually found the problem: the Trello client keeps around an anonymous object called `ready` that stores some session data. When a user clicks 'Deny', this `ready` object remembers. I was able to fix the Deny/Allow issue by pulling down a local copy of client.js and making an interface change to the module:

``` javascript client.js
clearReady: function() {
  ready = {}
}
```

From [another issue I was having](/blog/2014/07/07/deauthorizing-token-with-the-trello-client/), I already call `Trello.deauthorize()` before my application attempts to contact Trello, so now I also make a call to `Trello.clearReady()`. My `authorize()` function looks like this:

``` javascript trello-controller.js
var authorize = function (expires, callback) {
  Trello.deauthorize();
  Trello.clearReady();

  Trello.authorize({
    name: "Ollert",
    type: "popup",
    interactive: true,
    expiration: expires,
    persist: false,
    success: callback,
    scope: {
      read: true,
      write: true
    }
  });
}
```

Since I've pulled client.js locally, I'm no longer sending my developer token when I load the file. I manually set my key when the page loads, but this could just as easily be done in my `authorize` method above.

``` javascript layout.haml
:javascript
  $(document).ready(function() {
    Trello.setKey("#{ENV['PUBLIC_KEY']}")
  });
```

You can find a copy of my modified client.js [in this Gist](https://gist.github.com/larryprice/1e67ddcd53c686fbc1de).
