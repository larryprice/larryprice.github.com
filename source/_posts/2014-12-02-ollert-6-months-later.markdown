---
layout: post
title: "Ollert - 6 Months Later"
date: 2014-12-02 18:54:12 -0500
comments: true
categories: ollert trello non-technical
---

[Nine months ago](/blog/2014/03/17/sep-startup-weekend-ollert) I brought a team of 6 together to build a data analysis tool called [Ollert](https://ollertapp.com). [A few months later](/blog/2014/07/13/ollert-reveal-the-data-behind-your-trello-boards), I started working with [SEP](//sep.com) to find time to make Ollert better. Where do we stand today?

From a traffic perspective, we're doing pretty well. The numbers are increasing steadily over time, as are the number of people who use the application on a near-daily basis. Although we have surpassed our modest initial goals, we would like to build up a larger audience in the coming months.

We've had about 1800 different users visit the site for a total of over 5000 pageviews. On average, users hit about 2 pages per visit and stay with us for almost 90 seconds at a time. About half of all traffic comes from social media such as Twitter, LinkedIn, and Reddit, but we've also seen some interesting traffic patterns come out of Trello, [my blog](/), [SEP Labs](//sep.com/labs), and various other blogs.

[Trello](//trello.com) reached out to us a few months ago to congratulate us on making such a neat application using their API. They sent us some shirts and stickers for our trouble. It was an extremely satisfying moment.

On the technical side of things, we've made several changes to the application.

We removed the concept of "signing up" for Ollert, as it was the greatest cause of confusion on the site. I replaced it with a mechanism that "automatically" signs a user up when they connect to a Trello account using OAuth. This makes our lives easier because we don't have to store passwords, and the lives of users easier because they don't have to come up with and remember another password.

Every aspect of the site has been made faster. Server requests have been made faster by switching to unicorn and utilizing the "workers" feature. Trello API call times have been slashed - Board fetch time dropped from 1.8s to .18s, Label chart 17.6s down to .426s, WIP chart 18.2s to .221s, CFD 15s to 1s.

We added two highly-requested charts - the Burn Down and Burn Up. It takes a certain type of Trello board to make these useful, and I'm looking forward to finding someone one day who's able to make good use of them.

We also improved the look of all pages (most notably the home page) with some help from SEP's UX team.

Oh, and now it works significantly better on mobile.

What does the future hold for Ollert?

In the short term I'm looking to implement a Cycle Time chart and add in a baseline for the Burndown chart. Long-term is a little less clear, but we may have a sizable announcement soon.
