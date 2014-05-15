---
layout: post
title: "Takeaways From AgileIndy 2014"
date: 2014-05-14 20:59:00 -0400
comments: true
categories: retro non-technical agile
---

[The AgileIndy Conference 2014](//agileindy.org/conference/) has come and gone. I wasn't sure what to expect, since it was my first conference, but I came out of it feeling rather positive.

A short list of my favorite things from this year's AgileIndy:

* Interesting speakers
* Good food
* Uncomfortable chairs
* [#agileindy14](//twitter.com/search?q=%23agileindy14&src=hash)
* Free booze
* Lots of coffee
* Champion dogs

Good times. Personally, I found the first half of the day to be superior to the afternoon. I'm not sure if it was the speakers or if I'm just a morning person, but I'm much more interested in discussing what the morning speakers had to say.

###The Dude

> We've got hard problems to solve, we don't need aphorisms, we don't need truisms.
> -- David Hussman

The event started out with the self-proclaimed "Dude," [David Hussman of DevJam](//devjam.com/). I've previously had some training involving Mr Hussman at [SEP](//sep.com), during which we had trouble explaining our relationship with Agile: gargantuan project, lots of devs, massive codebase, hour-long builds, and an immature implementation of the so-called Scalable Agile Framework. I thought I knew what to expect during this talk, but I was delightfully mistaken. Everything Hussman said during his keynote fit with the conversations I've been having with my current teammates.

Hussman talked about "process mass," or the weight that you carry as you gain more and more process on a project. "Measure your success by evidence, not by adherence" to the process. What good is completing a story if you don't have anything to show for it, or if you don't have the quality to back it up? Hussman discussed scrum, and the ridiculous concept of "scrum of scrums," and potentially even a "scrum of scrum of scrums." In his own words, "At some point you have to have something other than a naming convention." Getting stuff done and feeling satisfied with your work is much more important than being able to audit the lifetime of a story, or what day of the week your team gets the most done, or how many meetings your ScrumMaster goes to.

Speaking of ScrumMasters (and Product Owners and Tech Leads for that matter), the Dude wants us to think critically about these roles. Why are Scrum Masters and Tech Leads in meetings all day? A Tech Lead mosies into standup in the morning just to say, "Yesterday I was in meetings all day, today I'm in meetings all day, no blockers." "Somehow [we've] bonded the process to the people," using process to hold team members hostage. We've found ourselves turning Agile into a bloated mess, and we're in need of people to relieve the pressure.

###Estimation

During the first break-out session, [Steve Vance](//vance.com/) talked about estimation, one of my favorite topics to complain about.

We started out by defining terms. Unfortunately, we use the word "estimation" to mean almost anything, from the very precise to the very vague. We "estimate" in T-shirt sizes, story points, days, and hours. We all mean something different when we make an estimation; it all boils down to an opinion. "The work is going to take what it takes," and everyone has an opinion on how long that is.

I mentioned hours in the above paragraph as a unit of estimation. This is a particular pet peeve of mine: I absolutely despise estimating in hours. What do we even mean when we estimate in hours? Vance says, "When I say hours as in this is going to take 3 hours, that does not mean 3 clock hours." So what _does_ an hour mean? My current team uses a tool called TFS, which actually forces you to answer this question. So we "estimate" that Danny Developer will be able to accomplish 5 hours of work every day. We just turned Danny's day into hours on a clock. Now we "estimate" that a task will take "about a day," which translates into 8 hours. Note that `1 task day != 1 Danny Day`. We're estimating tasks in these very precise units (real-life hours), and poor Danny will probably just work a 10-hour day to finish the task in "1 day," which is what the team estimated the task would take.

T-shirt sizing or story points should be able to alleviate these problems, but we have to be vigilant not to concretely define these terms. Estimates should always be relative to yesterday's weather and not based on real time frames. Once you've defined a 5-point story as taking exactly 5 days, you've destroyed the whole system and you might as well start estimating in hours again.

###Anti-fragility

[Si Alhir](//salhir.wordpress.com) discussed the principles of "anti-fragility" during the pre-lunch session. Anti-fragility is a post-Agile concept. That's right: _post-Agile_. I know you were just getting used to Agile, but it seems that we've created a monster that's getting harder and harder to control. Teams find themselves saying "They're not Agile, we're Agile" when describing the pitfalls of other teams.

So what is anti-fragility? It's easier to define in relative terms. Traditional teams are hurt by change and actively resist change; Agile teams embrace change and are able to inspect and adapt to it; Anti-Fragile teams embrace disorder and are able to adapt and evolve to chaos. "Fragile teams want tranquility," while anti-fragile teams "gain from stress." Anti-fragility involves shifting from focusing on avoiding risk to focusing on overcoming risk.

"What have you removed?" Si asks, reiterating Hussman from earlier in the day. "More rules, more process causes more fragility." As our Agile teams add process, this becomes more evident. "Dependency," "the line between dev and test", and "centralized power" all cause fragility. Sprint cycles every 1-2 weeks cause a constant stress, which "will kill you, numb you." Anti-fragile teams prefer "acute stress with recovery time" caused by overcoming risk as it materializes. The so-called ScrumMaster of an Agile team is protecting the team from risk, contributing to the overall fragility of the team.

"The most fragile thing in the Agile world is teams," Si insists. "Consistent, co-located teams." Teams should be able to be disbanded and created at-will or "just in time" as the need arises. Teams are created to attack "focal points," or areas that need attention right away. This is an especially hard pill to swallow, since many of us like the familial feeling that comes with long-term teams. Not to mention the nightmares of billing if you're a consulting company.

###Conclusion

There were a few other speakers at the conference, but none hit quite as close to home as the three I've discussed. Overall a very interesting conference and a very fun experience. I'm excited to think about these ideas and I'd like to attempt to use some of them to make my teams more efficient and maybe even a little happier. It looks like the general flow is to move away from massive amounts of process, working more like Lean Startup or Anti-fragile teams.

A decade from now, we'll be living in a post-anti-fragile world wondering what on earth we were thinking in this world of rules, definitions, estimations, and process.
