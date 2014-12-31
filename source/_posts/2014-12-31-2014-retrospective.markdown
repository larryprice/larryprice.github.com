---
layout: post
title: "2014 Retrospective"
date: 2014-12-31 11:40:00 -0500
comments: true
categories: non-technical retro
---

Happy New Year, dear reader.

2014 was a fairly good year for me. I wrote a lot of code, read a lot of books, and published a lot of blog posts.

###Things I got paid to do###

2014 was a reaffirmation of career goals alongside the opportunities to work with several very different clients. In addition to working on cool stuff, I was promoted to Software Engineer 2. During the tail end of the year, I had the chance to work from home part of the week, which was a great experience. A sampling of my professional works this year:

* I wrapped up my time writing Qt/C++ for a touch-screen automated-guidance system embedded in heavy machinery. I spent the first two months of the year playing Tech Lead for a small team of misfits leaving the project during an organizational restructure. I had the opportunity to work directly with a project architect to redesign features and guide my fellow developers through the implementation.
* I worked on [Health2Wealth](https://h2w.cc), a ruby application created by some work friends to track Fitbit data and allow an administrator to award monetary rewards for reaching goals.
* I pitched, designed, and implemented a product called [Ollert](https://ollertapp.com), a Trello analysis tool written in ruby. Built during a 48-hour [SEP Startup Weekend](http://www.sep.com/labs/startupweekend/), I led a team of 6 to bring the project to fruition. SEP has fully embraced the product and I continue to work on [improving it](/blog/2014/12/02/ollert-6-months-later/) when not on project work. We currently get about 300 sessions a month, and I frequently get emails from users thanking me, reporting bugs, or submitting feature requests.
* I became part of a team of developers working on a modern asset-tracking tool to replace an existing Microsoft Access tool. This was my introduction to C# programming with ASP.NET MVC. The project was short-lived for my team, but sparked my interests in RESTful APIs and Javascript.
* I worked on a rewrite of a huge WebForms e-commerce site. We created a hybrid site running some pages in the WebForms engine and all of our new pages in fresh MVC. We worked with a high-fidelity prototype created by an outside company, which proved at times invaluable and at other times a thorn in our sides. We worked directly with members of the client on-site, who were able to provide us with great insights into the legacy system and helped us work out unclear requirements.
* In December, I had some fun working on a solo proof-of-concept with a new client. This client is planning to do a rewrite of a large application in 2015 and wanted to do some investigation into rules engines, particularly those using Java. I worked through a few libraries and landed on JBoss Drools, an open-source rules engine with an integrated frontend called KIE Workbench. It's been a while since I wrote any Java code, and I'd certainly never deployed any Java web applications. After two weeks of work, I was able to launch a web application using Spring MVC and integrate a KIE Workbench instance running on the same Azure virtual machine. The application is able to pick up rule changes from the Workbench automatically, without recompilation or redeployment.

###Of my own volition###

I had a lot of fun coding ideas this year, most of them lost to the raptures of time. A few items of note:

* BYO Game of Life (http://byo-game-of-life.herokuapp.com)
  * My most recent release - a frontend web client allowing the user to give a URL pointing to a Game of Life backend. I created a [Go implementation](https://github.com/larryprice/game-of-life-impl) as part of my presentation for the inaugural [Indy Golang meetup](http://www.meetup.com/Indy-Golang/events/219199103/).
* 3rd Day Organics
  * We finally finished version 1.0 of 3DO, a custom e-commerce website for a co-worker's spouse's cooperative food program. We built the whole application from scratch; in hindsight, we should have set something up using [spree](https://spreecommerce.com) or another commerce framework.
* Fortune Cookie API (http://fortunecookieapi.com)
  * A fairly silly API written using NodeJS to get data associated with fortune cookies: fortunes, lessons in Chinese, and lottery numbers. I implemented a simple application to access the data at [http://demo.fortunecookieapi.com](http://demo.fortunecookieapi.com).
* mongoose-simple-random (https://www.npmjs.com/package/mongoose-simple-random)
  * My foray into NodeJS packages - this is a plugin which adds `findRandom` and `findOneRandom` methods to any mongoose schema. Used extensively in the Fortune Cookie API.
* Ollert (https://ollertapp.com)
  * I know I listed it under a paid project, but I was putting over 8 hours a week into Ollert between the time it was originally built and the time I was paid to work on it. It's my big pet project, and I'm proud of the things that it and I have accomplished. Trello has noted our accomplishments, validating all the hard work and long hours I'd sunk into it.

###Book It - Where's my free pizza?###

I think that I read 11 books this year. See my [books section](/blog/books) for a quick check of which books you should be reading. Some of my favorites:

* [Rework](http://amzn.to/14coDAd) and [Remote](http://amzn.to/1rzTsZH) by Jason Fried
* [Punished by Rewards](http://amzn.to/14coIUG) by Alfie Kohn
* [The XX Factor](http://amzn.to/1tAMWU0) by Alison Wolf
* [The Lean Startup](http://amzn.to/1Ai3mBk) by Eric Ries
* [Made to Stick](http://amzn.to/1x4jaXh) by Chip Heath

###This blog o' mine###

In reality, I put quite a bit of work into this blog.

I published __24 blog posts in 2013__; I published __37 in 2014__ (including this one).

I had __1700 sessions and 2420 pageviews in 2013__. I did almost that well in a single month at the end of 2014. I had __10,194 sessions and 12,997 pageviews in 2014__ (as of 10:46AM EST 12/31/2014).

In February, I set up a custom domain at "larry-price.com". Since then, [I moved from Github Pages to OpenShift](/blog/2014/12/14/migrating-an-octopress-blog-from-github-pages-to-openshift/) and added SSL.

I integrated Google AdSense this summer. Full disclosure: I've "earned" $3.78 in the past 6 months. From my perspective, these ads are simply an experiment to see how I could potentially monetize this blog. Another experiment was posting Amazon Associate links with my [write-up posts](/blog/categories/write-ups); I've gotten 4 clicks and 0 buys.

In an attempt to keep people on the site "longer", I added a Related Posts section above the Comments - it makes the blog take forever to generate and has had negligible results. It turns out users are more likely to click the related tags than the related posts. I intend to remove the Related Posts section soon (it may not be there when you read this).

###Hello, 2015###

As I enter the new year, I want to be able to look back next year at a set of naïve goals and wonder why I ever thought they were practical. So here they are:

* Contribute meaningful code to several open source projects not created by me
* Read at least 1 book per month
* Attend more meetups
  * As a note, I am in fact _starting_ a [meetup for Google Go in Indianapolis](http://www.meetup.com/Indy-Golang) starting on January 6, 2015. This goal may actually be attainable!
* &gt;3000 sessions/month on this blog (currently ~1400)
* Become pro at using chopsticks
  * I'm already pretty good, but by the end of 2015 I will be eating pizza with chopsticks
* More effectively conceal emotions when working with peers and clients
  * Use this energy and passion to put the situation in my control instead
* Work remotely more and better
* Obtain more technical leadership roles on projects
  * Not entirely in my control, I know, but I'm hoping that 2015 is a year where I'm able to demonstrate both my technical skills and proclivity to command

###Goodbye, 2014###

Thanks for reading. Have you thought about doing one of these yourself? You really should! There's lots of help out there if you need it - you can even reach out to me, if you like.

As always, may your compile times be short and your error messages meaningful.
