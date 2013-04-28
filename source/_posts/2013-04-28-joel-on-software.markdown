---
layout: post
title: "Joel On Software"
date: 2013-04-28 11:38
comments: true
categories: write-ups
---

> We're programmers. Programmers are, in their hearts, architects, and the 
> first thing they want to do when they get to a site is to bulldoze the place 
> flat and build something grand. We're not excited by incremental renovation: 
> tinkering, improving, planting flower beds.
>
> -- Joel Spolsky, "Things you Should Never Do, Part I"

###The Gist

_Joel On Software_ is a collection of blog posts by the great [Joel Spolsky](http://www.joelonsoftware.com/). He has risen to Internet Fame after his experience working for Microsoft and eventually starting his own company. The "chapters" (blog posts) provide insight into picking a language for your codebase, recruiting engineers, managing programmers, fixing bugs, paper prototyping, and how Microsoft works.

###My Opinion

Joel Spolsky is a god sent down from Mount Olympus to calmly explain to us why we are sometimes terrible people and then goes on to tell us it's okay and we should learn from our past. I loved this book, especially since I read it at the same time as some really terrible books (namely [The Inmates are Running the Asylum](http://www.amazon.com/Inmates-Are-Running-Asylum-Products/dp/0672326140) and [Enterprise Games](http://www.amazon.com/Enterprise-Games-Mechanics-Better-Business/dp/1449319564); do not buy or read under any circumstance).

Spolsky talks a lot about specs in the first part of the book, and at first I thought that he was stating too much of the obvious; but then why don't we do specs in this obvious way? Instead, we resign ourselves to write a tiny portion of the spec, start programming, and then implicitly change the spec while only haphazardly updating our documentation. I've been on projects before where we would have greatly benefitted from having a solid spec; we knew exactly what needed to be done, because we were rewriting a successful product the company had released several years ago. With an ever-changing spec, we programmed ourselves into a hole, and struggled for at least half a year to claw our way back out. I have to wonder if more pre-planning would have saved us from ourselves.

Spolsky's advice on hiring people is invaluable ([The Guerrilla Guide to Interviewing](http://www.joelonsoftware.com/articles/GuerrillaInterviewing3.html)). In fact, I wish that I had read some of these blog posts while I was interviewing. Spolsky says that recruiters should ask "impossible questions" and not worry about what the answers given are, just that the recruit comes up with something. He says to stop asking trivia questions. He talks about relying on all interviewers equally: if any two interviewers say "No" to a candidate, the candidate is toast. He talks about asking candidates about projects and looking for passion, which is almost always the sign of a good programmer.

Spolsky talks about the commodity market of software/hardware. To sum it up, the supply of one drives the demand of the other. This section made me think about the current PC market: hardware is cheap _and_ software is cheap. Not only is software cheap, but it's often _free_. Prices for hardware haven't been going down at nearly the rate it used to, but that seems natural as the price approaches the cost of labor and parts. Most of the hardware innovation right now seems to be aimed at the tiny computers in our pockets (or [on our face](http://www.google.com/glass/start/), soon enough). Google's software makes money by displaying ads using its search engine, so Google has entered the market of making software that is compatible with all kinds of different hardware. Google has played the commodities game like a champ, getting hardware running their software into hundreds of millions of pockets.

Some of the weaker parts of the book are when Spolsky talks about .NET. I'm not going to go into detail, but when he wrote these posts, .NET was just entering the market. Spolsky starts by bashing .NET for making all the old Windows API code unusable, but in later blog posts falls hard for .NET and starts thinking about ways to rewrite his entire life in .NET.

###Who Would Like This

I believe that programmers are the main audience for this book, and I am a programmer who loved reading it. Spolsky does not talk down to the reader, nor does he dumb down the nitty-gritty technical blog posts.

Programmers who intend to become managers may get added benefit from this book, as Spolsky discusses some interesting recruiting techniques and ways to deal with programmers. My company uses some of Spolsky's advice when hiring engineers (quite a bit of it, thinking back to my interview), and we've been hiring only the best and brightest cowboys in the Midwest for years.
