---
layout: post
title: "Everybody's Talking About Whiteboard Problems"
date: 2016-04-30 22:00:00 -0400
comments: true
categories: non-technical
---

Interviews suck, amirite? Recently, there's been a rise in posts discussing how harmful whiteboard problems are. Let's chat about how the internet works, and then we'll discuss where I land on this issue.

You've got one group of people who love whiteboarding. Whiteboarding allows interviewers to see how a candidate thinks. Whiteboarding gives the illusion of programming without the fear of compiler errors. Solving a problem algorithmically on a whiteboard is the same as programming.

And you've got the people who think whiteboarding during interviews is just as terrible as waterboarding a candidate. Whiteboard problems cause interviewers to become smug, sexist, and xenophobic. Candidates are forced to sweat bullets at a whiteboard while a death panel of interviewers silently judge them. Whiteboarding a programming problem is entirely different from computer programming.

Between you and me, I don't believe it's all so black and white. Like most things in this world, whiteboarding falls into a somewhat uncomfortable grey area. There are certainly things which can be gained by whiteboarding, but it's also in no way indicative of programming strength or culture fit. There are literally classes teaching people how to be pro at whiteboarding.

One of the main problems is that this is not how you work in real life. You don't put together code on a whiteboard while a group of strangers stares intently waiting for you to screw up so they can mark points off your score. Your day is largely spent reading other people's code to figure out how you can force whatever new feature or bug fix you need to implement into their code as easily, quickly, and cleanly as possible, and then you're usually put into a position where you have to sacrifice the easy, quick, or clean part of that plan. Along the way, you usually get to fight the compiler or search the internet for any relevant APIs, StackOverflow questions, or existing libraries. You're probably also a member of a team with varying experience levels in the current codebase, so you'll spend another big part of your day asking/answering questions on group chat and gatekeeping the codebase from code that is unreadable, unchangeable, or just plain questionable via code reviews.

But if this is the way we do our jobs, why do we ask candidates to reimplement underscore's debounce function, create all the possible words from a telephone number, or generate roman numerals from an integer?

Some organizations have moved to systems of making the candidate solve a multi-hour problem in their free time, or come into the office and spend a day (or more) pairing with their future teammates. Although I think these ideas are founded on a solid base of logic, they are terrible. A large number of worthy candidates already have full-time jobs to deal with, and many of them have other commitments during the evening hours. The perfect candidate may have recently found themself with a new child, and every non-9-5 hour is no longer available to create a reverse binary tree. Adding more work and stress to an interview is not the right way to go.

How could we possibly find the perfect candidate without stressing them out and giving them standardized tests?

__Let's make them work like we do, but only for a reasonable amount of time.__ Come up with a medium-difficulty coding problem and _write it yourself_. But don't make it clean. _Make it sloppy_. Make _a ripe mess_ of that code. Add an obvious bug or two.

Now, when you bring in an interviewer or talk to them on a video call, show them this code. _Can they figure out what it does_? _Can they find the bugs_? Ask them to code review it. _Do they see all the terrible things you've done_? _Can they tell you how to make it better_? _Can they do it without being condescending_ (news flash: most full-time programmers cannot do this)? This should all be a back-and-forth, with the (lone) interviewer working with the candidate to discuss any issues. If you have time, ask them to actually correct and refactor some of the code. If you're into that kind of thing, work with them to write some test cases for the bugs you've fixed. If for some reason you still don't know whether or not you like this person, have them add some functionality to the codebase. Then code review their code and see how they handle your suggestions. __You should limit this exercise to an hour__, maybe two if you're a sadist. Do not make this an all day thing.

Think about how much you could get out of this exercise. You'll determine whether the candidate knows what code looks like, knows how to read code, knows how to communicate about code, and knows how to fix bad code. You'll also get a bit of their personality based on how mean they are during the code review sections. Hopefully, you were able to have a real discussion about design decisions and coding during the interview. The candidate will have done very little programming, just like you and me on a normal day. Best of all, you'll only have spent an hour of each other's time trying to figure out if the candidate is competent. Leave time for questions at the end and be done with it.

Of course, I don't currently run a tech company or do much with the interview process. But everyone's been giving their two cents on that issue, and I like to type words on the computer. What do you think? Is whiteboarding during an interview integral to your interview process? Do you need it? Do you hate it? Do you have any alternatives? Let me know in the comments.
