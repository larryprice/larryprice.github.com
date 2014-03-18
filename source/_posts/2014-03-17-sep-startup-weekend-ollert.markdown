---
layout: post
title: "SEP Startup Weekend: Ollert"
date: 2014-03-17 20:07:38 -0400
comments: true
categories: web trello ollert
---

Last weekend was [SEP](//sep.com)'s 6th Semi-Annual [Startup Weekend](//sep.com/labs/startupweekend/). For those unfamiliar, software developers pitch ideas Friday evening and developers volunteer their time to come up with a [minimum viable product](//en.wikipedia.org/wiki/Minimum_viable_product) in the next 48 hours. Free beer is the only thing that makes such a weekend possible.

I've been before and participated in other people's projects and it's always a blast. However, this weekend was different: I had an idea to pitch. Although the idea went through many names, the core concept remained the same:

> To tell Trello users what their boards say about the past and the future through unique visuals.

[Trello](//trello.com) is a collaborative workflow management tool that does a wonderful job of showing you the present. There is currently no way on Trello to see your past history or compare yesterday's weather. This simplicity is part of the beauty of Trello, but also an opportunity.

My idea was to create a web application where a user could quickly and easily connect with Trello and view information he or she had never seen previously. I would offer a trial service requiring no login that would allow access to all this data, given that the user puts up with authenticating with Trello every time he or she visits the site. There would be a free membership, which would allow the user to "permanantly" connect to Trello. To monetize, I wanted to offer a paid membership, where the user would be given the ability to compare "historical" Trello data by selecting begin and end dates for the Trello data that is analyzed.

[Ollert][ollert] is the result of this Startup Weekend idea. A live version of Ollert can be found at [ollert.herokuapp.com][ollert].

I worked on [Ollert][ollert] with 5 other great developers, and we got a spectacular amount of work done given that we only spent a single weekend programming. We were able to direct users to connect with Trello, let them select a board, and then generate and display 12 different statistics and analyses. We also implemented Sign Up/Login.

We worked on [Ollert][ollert] to the last minute, so not everything got in. We never implemented the paid member feature and we didn't get in all the analytics we wanted. We also had some great ideas come out while we were working on [Ollert][ollert] that didn't make it into the application, such as filtering chart types and selecting favorites.

Overall, my teammates and I had a great time and we are confident that we've created something useful.

My current intention is to do several more blog posts about [Ollert][ollert] including Connecting to the Trello API, Using ruby-trello, Using sqlite on Heroku, What I Should Have Had Ready Before Asking People To Work For Me, and The Future of Ollert.

[ollert]: //ollert.herokuapp.com

