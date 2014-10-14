---
layout: post
title: "The wisdom of Confucius - now available in an API"
date: 2014-10-14 06:03:16 -0400
comments: true
categories: javascript node mongo api web
---

I make a lot of silly things while learning new technology. In this case I have tapped into the almighty power of [Confucius](https://en.wikipedia.org/wiki/Confucius), a Chinese philosopher from 2500 years ago credited for writing or editing many Chinese classic texts. Growing up as an American child, I associated Confucius with proverbs, and I associated proverbs with that classic after-Chinese-dinner dessert [the fortune cookie](https://en.wikipedia.org/wiki/Fortune_cookie).

Fortune cookies are a pure delight: a message in a bottle from the restaurant's proprietors for you to enjoy as you kindly vacate the facility.

Most fortune cookies that I've opened recently have three parts: a fortune (generally a proverb), a lesson in simplified Chinese, and a lottery number. So many secrets wrapped up in such a small, golden treasure.

Ever left a Chinese restaurant, gotten in your car, and felt the need to break something only to realize you left your precious fortune cookie on the table? I have built a solution.

[The Fortune Cookie API](http://fortunecookieapi.herokuapp.com/) is a simple, RESTful API built to generate fortune cookie data. The root URL shows documentation built using [Apiary.io](https://apiary.io/).

How does it work? I need fortunes now!

There are options. You can get `fortunes` from the [/v1/fortunes](http://fortunecookieapi.herokuapp.com/v1/fortunes) endpoint, `lessons` from the [/v1/lessons](http://fortunecookieapi.herokuapp.com/v1/lessons) endpoint, and lottery numbers from the [/v1/lottos](http://fortunecookieapi.herokuapp.com/v1/lottos) endpoint. By default you get 100 of any model, but all endpoints include a `limit` (max 1000), `skip`, and `page` parameter to facilitate getting all the lessons and fortunes. For lottery numbers, we approximately build [Powerball](https://en.wikipedia.org/wiki/Powerball) numbers except we currently ignore the rule for red balls, which means there are something ike 42 billion different possibilities. Due to the high number of potential lottery numbers, the `lottos` endpoint also includes a `firstId` parameter that lets you start from anywhere.

But there's no need to get the individual models (unless you're into that kind of thing)! I also created a [/v1/cookie](http://fortunecookieapi.herokuapp.com/v1/cookie) endpoint to retrieve a random fortune, lesson, and lottery number as a single object. Woohoo! You can specify the number of cookies (max 100) with the `limit` parameter.

```
GET http://fortunecookieapi.herokuapp.com/v1/cookie

{
  "fortune": {
    "id": "53ffcf1d4ea4f76d1b8f223e",
    "message": "This fortune intentionally left blank"
  },
  "lotto": {
    "id": "001000200030004000500006",
    "numbers": [10,20,30,40,50,6]
  },
  "lesson": {
    "id": "53ffcf1d4ea4f76d1b8f2241",
    "chinese": "因特网",
    "pronunciation": "yintewang",
    "english": "internet"
  }
}
```

Now you can fill that hole in your heart where the fortune cookies are missing. If you're interested in the code, you can [check it out on Github](https://github.com/larryprice/fortune-cookie-api).
