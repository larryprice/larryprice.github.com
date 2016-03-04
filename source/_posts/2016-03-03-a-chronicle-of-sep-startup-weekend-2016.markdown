---
layout: post
title: "A Chronicle of SEP Startup Weekend 2016"
date: 2016-03-03 15:16:29 -0500
comments: true
categories: retro javascript react web
---

Last weekend was the 2016 SEP Startup Weekend, a bi-annual hackathon where a few of the engineers get together for 48 hours to build things. As far as most people know, we transform massive quantities of beer, coffee, and unhealthy food into cohesive piles of code. Although that about sums it up, I thought it might be nice to chronicle my team's experience of this season's event. Enjoy!

### The Pitch

In my first startup weekend since [Ollert](/blog/2014/07/13/ollert-reveal-the-data-behind-your-trello-boards/), I pitched an idea for an instant runoff voting platform called RePoll. RePoll would allow users to create polls, rank candidates, and view results in a login-less system via a web portal, Android, and iOS app. To make it all interesting, I wanted to do all of the coding in javascript by using [React](http://facebook.github.io/react/) for the web frontend and [React Native](https://facebook.github.io/react-native/) for the iOS and Android apps.

### Friday

I somehow convinced 3 of my coworkers to help me on Friday night, who I'll refer to as H, G, and K, forming teams by combining the letters.

After sending around a few images and documents with my proposed architecture, I had GH start working together to build the Android app in React Native. Unfortunately, we hit a few snags in getting React Native to run an Android app on actual hardware, but late into the evening everything was up and running on a [genymotion simulator](https://www.genymotion.com/). By the end of the night, GH had a good portion of the [React Native opening tutorial](https://facebook.github.io/react-native/docs/tutorial.html#content) completed.

In the meantime, K and myself were hard at work building a core web API in NodeJS for all apps to use. We decided to use Mongo so we didn't have to worry about updating schema and initializing databases, and we used [Postman](http://www.getpostman.com/) to test our API calls. I may have overarchitected the session/token-based authorization system, but we got over 50% of the way done with the API. In my initial estimates, I wanted to have the API 100% completed by Friday night, but we were all tired and decided to break for the night around 10p.

Although everyone else headed out the door, the garage was packed with cars leaving a fancy-pants event held near our office. I stayed behind for about an hour setting up our API to run on [heroku with a MongoDB instance](https://re-poll.herokuapp.com/), and then configuring DNS for the domain I had already purchased.

### Saturday

I arrived early and laid down [some sweet jams](https://open.spotify.com/album/0XxLqWttW6u6vP3Yz9Sye3). I continued working on the API alone and had K start working on the iOS app. We added a new teammate, dubbed A, who started work on the web frontend. The API neared completion by the end of the afternoon, and the mobile apps were beginning to hook into the publicly accessible API. During this time, several bugs were found in the initial API code and were fixed. The iOS app was making quick UI progress, but having issues connecting to the API. I started floating between dev teams as I got drowsy, but eventually got a second wind and started helping with the web frontend. The Android and web applications were both able to create polls before everyone went home. A started using a web React datetime component to set start and end dates, while the Android team was figuring out how to use the native calendar and clock to select dates and times.

To store data, the mobile teams started using the [Async Storage](https://facebook.github.io/react-native/docs/asyncstorage.html) library which allows for OS-agnostic storage of data in iOS and Android. On the web frontend, we used local storage to keep around any information we might need.

Near the end of the night as I started working on the poll results page on the web frontend, I started finding system-crashing bugs in my interpretation of instant runoff voting, and eventually found that the results weren't always correct.

One item slowing us down Saturday when writing React Native code was its insistence on using syntax defined in ES2015. Our team was largely unfamiliar with this new syntax, so this tripped us up a bit more than expected. All in all, it was a great learning experience to see this cutting edge specification in action.

### Sunday

Early again, I created a [heroku app for our web frontend](https://repoll-web.herokuapp.com/) and set up the DNS appropriately. As the rest of the team started to pile in (and the smells of bacon arose from the Commons), I transitioned to trying to fix the incorrect ballot-counting logic in the API. I ended up rewriting the code several times, but finally found an algorithm that worked correctly. At this point I was kicking myself for not writing tests to verify we were writing good code.

The Android team caught a second wind, finishing the create poll page and started tackling poll lookup. The original plans called for a typeahead, which ended up being tougher to implement than expected, so the Android app instead supplied a simple textbox to attempt to match an existing poll. In doing this, the Android team successfully created an authentication token in the API and was able to display poll candidates in the app before running out of time.

The iOS team continued to have issues with the API, but successfully mocked out most of the rest of the app right before the demos.

Due to familiarity with the tech, the web team was able to get most of the way to a completed layout. I hastily fashioned a dual-list system for seeing candidates and ballot selection, while A transformed that system into a drag and droppable component. We were able to submit updated ballots to the server and fetch previous ballots for our poll tokens. With the updated API, the results page started working and it became possible to view the steps involved in eliminating candidates during the runoff process.

During demos, we presented all three apps and everyone was quite impressed at our javascripting.

### Action Items

* _General Things_
  * __Learn more about React Native.__ I was largely shielded from the pain as I was working on the API and React web parts of our project, but from what I've seen and heard React Native is an incredible framework for getting things done and sharing code and code paradigms.
  * __Start using ES2015.__ Tools like [Babel](https://babeljs.io/) allow us to start using next-generation javascript standards to write code now even if the browser support is unavailable.
  * __Do more with [webpack](https://webpack.github.io/) or [browserify](http://browserify.org/).__ I want to be able to use these tools to optimize pre-rendering on our site, but we were in such a rush we primarily used webpack to enable us to use several NodeJS dependencies. Still cool, but we there's so much else to explore.
* _Startup Weekend_
  * __Estimate better.__ I drastically underestimated the amount of work needed to get this project completed in a weekend. We would have had a better chance had I pre-built the API. I think doing some pre-work in this case would have been okay, but I also could have asked for more help. Next time I would like to do a better job of figuring out how much work there is and how much time it will take.
  * __Be an expert.__ About half the work we did this weekend was in unknown territory for me. I should have studied further on getting React Native working before the event, and I should have known a little bit more about ES2015 as well. Although the lack of expertise slowed us down a bit, my teammates were extremely adaptive and loved learning the new tech.
  * __Create and prep the team beforehand.__ I had enlisted one engineer before the pitches Friday night, and I ran my presentation through that engineer to verify it all made sense. I should have also run some of my architecture ideas past that engineer to get buy-in. I also wouldn't mind having the team mostly formed before we start, but that's a big commitment to ask of people.
  * __Take more frequent, shorter breaks.__ We hit the Monon Saturday afternoon and walked for 30-40 minutes which ended up making me drowsy. However, the fresh air did us some good and was revitalizing for the project in general. It would be great to have more frequent, 10-15 minute walks, but it's really difficult to find a time when the team is ready to take a group break in such a fast-paced environment.
