---
layout: post
title: "The Trouble with Time Zones (aka Learn From My Mistakes Part 1)"
date: 2013-08-07 22:00
comments: true
categories: retro
---

> As a user, my data's LastModifiedDate should not be updated when I Export it, and it should be in UTC.

That's the gist of the story I was tasked with a few weeks ago. Seemed easy enough: just set the "LastModifiedDate" field of each XML node to the LastModifiedDate pulled from the database converted to UTC. I can do the development and write up some Cucumber tests in an hour, two hours tops. Find a few small caveats along the way, get it out of development and through code review by the next morning. Push it to the mainline and watch the build go green. All done, dust off my hands and move on to something else.

When has anything ever been that simple? Certainly not in the fantastical world of software development! About ten minutes after the build server confirmed a successful build of my code on the tip of our main branch, a face popped up over the cube wall. "Hey, why'd you break the build?" My teammate was having trouble running my LastModifiedDate tests. All of his time comparisons were off by an hour. My computer (and the build server) were in one time zone; his was in another. I immediately realized the exact lines of the code that weren't dealing with time zones. We agreed to let the code remain on the build server instead of backing it out since the build server was green, and I created/tested/pushed a couple of patches to fix the problem. Green build on the build server. Green build amongst all my time-disabled peers. The next morning I had a panic attack in the shower thinking about how some of my test data could be affected by the switch to UTC. I rushed into work early and put in another patch, this one slightly changing the format of some of my test data. After doing this, I realized that what I had done was completely unnecessary and decided I should probably get my blood pressure checked.

Then: silence. I was satisfied I'd seen the end of my time zone issues which I'd spent nearly as much time patching as I had developing the initial solution. Two weeks pass. Monday morning there's an email from another country in my inbox, "This test was written that only works in American time zones! Who is fixing this?" After quite a few emails and chats I found that their main branch had merged in a version of our main branch between my original solution and my patch. The developer agreed to be patient until his team could start using the newer version of our branch. Four hours later I get another email from developers on another continent. I sent them the patches I made previously compressed into one to keep them working.

Was this a small mistake? On a project with a small, local team the answer is definitely yes. This project in particular involves a large number of developers who don't all test in the same time zone and who work on a number of different branches that feed into each other in weekly chunks. Despite that, I think this error could have gone greatly unnoticed had I simply backed out the original changes and pushed the patches and original solution together. My confidence (hubris?) really bit me on this one. Initially only a single developer seemed to be affected by my oversight; fate selected a build between my original solution and my patches to push to the other branches to make sure more people noticed my mistake.

**In hindsight**, The Right Thing to Do was to assume fault and back out immediately. One developer having build issues is still one more than should have been affected. Here's hoping this bit o' hindsight today leads to better foresight tomorrow.
