---
layout: post
title: "Extreme Programming Explained"
date: 2015-08-02 20:14:49 -0400
comments: true
categories: write-ups
---

### The Gist

[_Extreme Programming Explained_](http://amzn.to/1KKM3Q0) is the _de facto_ book on XP written by Kent Beck. In the world of XP, software development is fun and efficient. Communication is open between all parties, pair programming is mandatory, builds are short, and tests are written before production code. Beck invites the reader to apply bits and pieces of XP to the development process to see incremental improvements, which is a very XP way of implementing XP.

### Takeaways

XP is about solving people problems. Nearly every challenge we face during software development is caused by dealing with people. Communication is key to fixing most of these problems. On a developer level, pair programming and colocation form closer relationships and cause more conversations to occur.

Do everything in small chunks. Whether you are designing a user experience, architecting a system, or estimating stories you should only address as little as _responsible_ at a time. This means architect a small system first and adapt as a more complex architecture becomes necessary instead of doing it all at once. Create a user interface for the next week's worth of work instead of the whole project. Small chunks means tighter feedback loops and less wasted effort when specs change (and they will change).

Software is built by people. People don't like to be referred to as "cogs in a machine." People make mistakes and people create (process) waste. People like to think they are contributing to the success of the company and they like to feel good about what they are creating.

10-minute builds. Beck claims that the perfect build takes 10 minutes; anything more makes people not want to wait for it to finish and anything less isn't enough time to take a coffee break. This concept is not for your constant, at-your-desk 10-second build. When I push my code to the server, it will compile a clean build and run every regression and integration and UX test within 10 minutes. If your regular, 100 times a day desktop build takes more than 10 minutes, you may want to see a shrink.

Teammates should have disagreements but not knife fights (at least not while on the clock). If there are no dissenting opinions about how to implement something you may never find the most efficient answer.

Automated tests are a man's best friend. Tests are a contract describing what the code currently does. Tests can be run by any programmer to verify a working system.

Did I mention test-first development? I've been through TDD trainings and found testing first difficult to follow strictly. The point of writing your tests first is to prevent any unwanted or undocumented side-effects in the code you just wrote. If you can delete a line of code and the tests still pass, that line of code is either unnecessary or untested.

Taking baby steps is okay. It's unlikely you'll be able to institute all the great changes recommended by XP. Start by figuring out how to get your 15-minute build under 10 minutes. "Force" your usually unwilling teammate to pair with you when working on a piece of code within said teammate's area of expertise. Take a piece of untested code and wrap it in tests.

### My Action Items

* Write more tests. I've been having a tough time getting tests written in this current project because it's largely UI and initializing a provided library. This week I want to come out net-positive on tests.
* Test first. I've accomplished this on a few occasions, including a recent bug fix I pushed to an open-source library I created. But this is hard. I have a tendency to dive in headfirst when a test could help set the stage for a simpler design.
* 1-week iterations. I have pushed against 1-week iterations because I often found them to be "too short," but I'm starting to come around to the idea of getting 1-2 items done per developer (or pair) per iteration. Other benefits include shorter feedback loop and less estimation required. My current project is a team of one, so this action item might have to sit on the backburner for a few months.
