---
layout: post
title: "Deauthorizing Token with the Trello Client"
date: 2014-07-07 05:59:13 -0400
comments: true
categories: ollert trello javascript web
---

In my [application](https://ollertapp.com), a user can connect to Trello without logging in. Whenever this "anonymous" user hits the landing page, I attempt to force the [Trello client](https://trello.com/docs/gettingstarted/clientjs.html) to authorize the user again. By doing this, the user can return to the landing page whenever he or she likes to switch usernames. My authorize code looks like this:

``` javascript
function AuthenticateTrelloAlways() {
  Trello.authorize({
    name: "Ollert",
    type: "popup",
    interactive: true,
    expiration: "1hour",
    persist: false,
    success: onAuthorizeSuccessful,
    scope: {
      read: true
    },
  });
}
```

This works oh-so-wonderfully in Chrome and Firefox, but, even during the hackathon which spawned [Ollert](https://ollertapp.com), we noticed that IE10/11 were causing some unexpected issues. Authorization would work the first time the user hit the landing page, but on subsequent visits telling Trello to Allow or Deny access resulted in the popup showing a white screen and never calling my callback function. Closing and reopening IE would allow me to authorize once, presumably until the "1hour" that I requested the original token for expired. I also verified this problem existed in IE9.

After several hours tweeting obscenities about IE, I stumbled upon the answer while browsing the source code for Trello's [client.coffee](https://trello.com/1/client.coffee). About one third of the way through the code, I found this function:

``` javascript
# Clear any existing authorization
deauthorize: ->
  token = null
  writeStorage("token", token)
  return
```

All this code does is unset the class variable `token` and unset the local store variable of the same name. So I changed my `AuthenticateTrelloAlways()` method:

``` javascript
function AuthenticateTrelloAlways() {
  Trello.deauthorize();

  Trello.authorize({
    name: "Ollert",
    type: "popup",
    interactive: true,
    expiration: "1hour",
    persist: false,
    success: onAuthorizeSuccessful,
    scope: {
      read: true
    },
  });
}
```

Voilà. Why does this only happen in IE? I was originally going to blame the local store, but, since I was able to reproduce the defect in IE9 (no HTML5), I no longer believe that to be the case. I'm currently resigned to chalk it up as IE just being IE.
