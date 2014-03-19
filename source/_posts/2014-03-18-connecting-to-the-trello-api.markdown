---
layout: post
title: "Connecting to the Trello API"
date: 2014-03-18 20:00:11 -0400
comments: true
categories: trello javascript html web ollert
---

[Trello](//trello.com) has a [pretty sweet API](//trello.com/docs/), which we use extensively in our Trello-analysis app [Ollert](//ollert.herokuapp.com). Initially connecting to the Trello API took us a few hours, so I"d like to make a record of how we managed to connect.

Making a connection to Trello requires two hashcodes: an application key and a Trello member token. You can generate and view your application key by visiting [https://trello.com/1/appKey/generate](//trello.com/1/appKey/generate).

The member token is something we need to get from the user. There are two ways to get a user"s member token: through fragments and through a __postMessage__. You can also request different levels of access (read, write, read+write), and different expiration periods (such as 1 day, 30 days, or never) for member tokens. For the remainder of this writing, I"ll be accessing a read-only member token that never expires.

We didn"t have a lot of luck with fragments, but the concept is simple enough. You have the user click a link that probably says "Connect With Trello" which is similar to:

`https://trello.com/1/authorize?key=applicationkey&name=applicationname&expiration=never&response_type=token`

At this point, the user is redirected to Trello and given the opportunity to Allow or Deny your application access. Once allowed, the user sees a static Trello page with their member token in plain text. Somehow you"re supposed to convey to them that they should copy this token and paste it back to you. This has clear drawbacks in usability.

Using the __postMessage__ method of accessing a member token was significantly more fruitful. Trello provides a Javascript file named [client.js](https://trello.com/docs/gettingstarted/clientjs.html) that does most of the legwork for you. An example:

``` haml
%script{src: "//api.trello.com/1/client.js?key=applicationkey"}

function AuthenticateTrello() {
  Trello.authorize({
    name: "YourApplication",
    type: "popup",
    interactive: true,
    expiration: "never",
    persist: false,
    success: function () { onAuthorizeSuccessful(); },
    scope: { write: false, read: true },
  });
}
function onAuthorizeSuccessful() {
  var token = Trello.token();
  window.location.replace("/auth?token=" + token);
}

%a{href: "javascript:void(0)", onClick: "AuthenticateTrello()"}
  Connect With Trello
```

When the user clicks the link, we have Trello set to activate a "popup" that will ask them to "Allow" or "Deny" our app from accessing their data. When the user allows us access, the popup closes and we hit the "onAuthorizeSuccessful" method. In my method, I simply redirect them to the `/auth` route with `token` manually added to the params list. One of the interesting options listed above is the "persist" option, which tells Trello whether it should be partially responsible for remembering your token. By telling it not to persist, the user will be presented with the popup every time. 

You can learn more about member tokens from [https://trello.com/docs/gettingstarted/authorize.html](//trello.com/docs/gettingstarted/authorize.html).
