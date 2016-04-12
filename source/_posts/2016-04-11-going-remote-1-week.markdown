---
layout: post
title: "Going Remote: 1 Week In"
date: 2016-04-11 20:40:00 -0400
comments: true
categories: non-technical remote
---

_I would like for this to be the first in a series of blog posts detailing my experience migrating from a culture based on co-location to a fully remote environment. Look out for another post at about the 1-month mark._

I've spent an entire week working remotely. How did it go? I'm not going to go into my current job here (if you're curious: I love it), but I'm going to try to focus on the aspects of setting up a pleasant environment for working from home and how I coped during my first week.

### But first: some background ###

I've never worked remotely as a professional software engineer for more than one day at a time until now. I've been a part of an office with traditional cube farms and an office with more modern team spaces. I haven't worked in a [Silicon Valley Sweat Shop](https://twitter.com/ptone/status/582764080219320323), and I've never had a private office with walls, windows, and that holy grail of office furniture: a door. My past organizations have been nested in a culture of co-location which is, of course, very normal. However, I've very recently moved to a position at a new organization with cultural values borrowed from the open-source community. About 70% of the 700 employees are full remote.

When I use the term "working remotely," I mean that a given employee works from wherever they want. For the majority of people it's their home or apartment, but it could also be a coffee shop, a co-working space, a public park, or anywhere with an internet connection and a nearby power source.

### Setting Up My (Physical) Environment ###

Also known as _Things I Had to Buy to Make This Work_.

To prepare for this new work environment, I did what anyone would do: I started buying things. I bought a [cool wireless mouse](http://amzn.to/1Q3HywG) that I could connect to my laptop through it's built-in bluetooth. I bought [a chair with a name from IKEA](http://www.ikea.com/us/en/catalog/products/00103102/#/90289172) and one of those cheap plastic carpet protectors. I started looking into motorized sit/stand desks, but didn't purchase one immediately (a mistake I quickly recovered from). I moved a second lamp into my home office and figured I was set up for success.

My first day, poor Markus (the chair) cracked the floor protector in several places, and my carpet is so thick that the chair couldn't roll on its own. I painfully used my laptop's keyboard for about 15 minutes before finding [another mechanical keyboard](http://amzn.to/1MqycR8) to use instead which arrived that afternoon. The height of my old desk made sitting for hours super uncomfortable, so I quickly put in an order for a [sit/stand desk](https://www.myupdesk.com/) that I could use downstairs where the carpet is thinner.

The next day, my new mechanical keyboard refused to let me type Shift+RightAlt without thinking it was a Meta key, causing all my keyboard shortcuts to be busted. I swapped it out with my old mechanical keyboard that worked just as well, but with a functioning Shift+RightAlt. It was around this time I noticed my WiFi connection was becoming flaky. I cursed my ISP and decided it was fine for the time being.

At some point during the week, the sit/stand desk arrived. Because of the thick carpet and uneven floors upstairs, I decided to start creating my new office downstairs. Unfortunately, my house is extremely poorly lit, so at the end of the day I had to run out to Target to buy a [new lamp](http://www.target.com/p/threshold-torchiere-floor-lamp-with-alabaster-glass-shade-oil-rubbed-bronze/-/A-14580986) (and some cream cheese! classic combo!).

By this time it was Friday. My network connection was not just bad, but worse than it ever had been. I started sniffing around the internets and found that [my wireless card](https://www-ssl.intel.com/content/www/us/en/wireless-products/dual-band-wireless-ac-7260-bluetooth.html) (a bluetooth/WiFi dual card) was notoriously bad at handling bluetooth and WiFi at the same time on Linux, but sometimes updating the kernel and getting the latest iwlwifi drivers fixed the problem. So I updated to 4.3. No luck, so I updated to 4.4. _Still no luck_. By the end of things, I had the 4.4.6 kernel and I honestly thought things were a little better. A few hours and a dropped Hangouts call later, I realized I was wrong. However, booting off of a USB with fresh Ubuntu gave me a consistent and fast network connection. After piddling around a bit, I dramatically smacked my forehead. I flipped off the bluetooth connection on my laptop, and my network connection was fast and stable. I had been thwarted by my beautiful bluetooth mouse which, fortunately for me, also came with the usual wireless USB dongle.

At the end of the week, I finally felt like my setup was fine. Network still a little slower than expected, but I fixed that recently by moving the router to higher ground.

### Let me count the ways... ###

OK, so I had some issues getting my home office in working order. But __I love working from home__.

I don't feel rushed when I get out of bed in the morning. I shave, shower, make coffee in the french press, and perfectly toast/cheese my bagel; and I get to enjoy every minute of getting ready for the day. I start when I want, but when I want is a normal time. I'm at my desk ready to do my job sometime between 7:30am and 8:30am. The only way I need to let people know I'm in the office is by signing into chat.

My house is almost silent except for the noises I make and the infrequent meow of a cat wanting attention. I can focus on what I'm doing without being distracted by peers or normal office noises. I don't need to put on headphones when I want solitude, which means I'm not distracted by whatever noise is coming out of the headphones. Achieving flow seems to be much easier when there are fewer distractions.

_Speaking of cats_... I know this will seem silly, but it's quite nice to have my feline companions hanging out with me all day. Usually they're taking naps nearby so I can say "d'awwwwwwww", or they're waiting in the kitchen for me to get up so they can get pets. Go [adopt a cat](http://www.adoptapet.com/cat-adoption) now.

I can make a healthy meal really easily at lunch. I love being able to pull some frozen veggies out of the freezer, throw them in the skillet with some sauce ingredients, add some protein, and feel like I'm eating well in the middle of the day.

All of my communication comes through chat, email, and infrequent Hangouts. We mostly just use Hangouts for our daily standup, and it's a great way to see everyone's faces and feel like a team. Email is pretty much only used for mailing lists, such as new merge proposals, bug reports, and company news; so I only glance at my email a couple times a day, and mark whatever I don't care about as Done. We use Google services to do this, which means I'm using Inbox to achieve zero-inbox. Chat is our primary means of communication. Being a remote group, the chat is extremely efficient. There are no graphic memes, and chats never end with "let's get a conference room to hash this out." Information is constantly being exchanged through the chat, and I rarely feel dissatisfied with the outcome. It's also interesting to note that when you leave the chat, you do not get to know what happened in your absence. However, this does not seem to be a hindrance to anyone, but almost more of a relief. You don't need to be in-the-know 24/7 to know what's going on.

### Near-Future Goals ###

* __Exercise more__. Although my eating habits feel a little better, my exercise habits have been much worse. I used to meander the office every hour or two to get 10-15 minutes of exercise, or go outside to walk around. I have my sit/stand desk set up now, and I shuffle back and forth based on what I'm doing and how my butt feels, but that doesn't count as exercise. I plan to lift my 10lb weights more frequently, and I'd like to work out my core with push-ups and bicycles. It would also be great to walk around the neighborhood in the summer.
* __Extracurricular__. It's been a while since I've worked on something meaningful in the evenings, and it's been tough to get started on something since my work office (and computer) is now also my play office. In the next few weeks, I'd like to build an Atom.io package to integrate with bzr.
* __Personalize workspace__. Since I've taken over the living room, the decor is mostly a big, decorative clock. I have some wicked posters I'd like to put up, but I need to convince my housemate that they won't look too tacky in such a guest-visible area.
* __Find a better floormat__. Like I said, this cheap IKEA floormat cracked on the first day. I'd like to find something vinyl or closer to rubber that will be more resilient but also more comfortable when I morph into standing mode.
