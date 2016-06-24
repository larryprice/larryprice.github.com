---
layout: post
title: "Ranks, Streaks, and Karma - A Look at Competitive Contributions"
date: 2016-06-23 21:15:17 -0400
comments: true
categories: non-technical
---

If you weren't paying close attention, you may not have noticed that Github has recently [removed Streaks from profile pages](https://github.com/blog/2173-more-contributions-on-your-profile) along with a few other changes.

For the uninitiated, Github tracked each user's commit history every day and displayed this information prominently on the user's profile for the past 365 days. When committing multiple days in succession, a "streak" is started. Your streak resets the next solar day you don't commit to Github.

__What was the purpose of the streak?__

The streak encouraged developers to contribute public open-source code as frequently as possible. This is a very good thing! Maybe it even influenced some people to contribute to open-source projects that desperately needed help. Maybe it helped some developers explore new codebases and languages they never dreamt of exploring previously.

However, any good feelings that came from the streak are instantly dashed when you take a day off. All those good-feels you get after working hard to get a 10-day streak? Crushed in an instant. Now you need to start over, but you don't bother because every time you feel a streak coming on it's dashed away. This is a painful reminder that life is short, and you'll only be rewarded in the short-term for any effort you put in.

__So if the good-feels wasn't the purpose, how about notoriety? Surely someone with a long streak is a good person and a noble developer!__

Except for [gists like this](https://gist.github.com/cirosantilli/4d24fc646ab9aec8def7) which will help you generate a 100-year streak. Or the fact that you could make empty commits to personal repos and manipulate the streak. Or you could be making minor documentation/newline changes every day. Not only that, but not all developers use Github as their primary source control platform. Developers could be making contributions to Bitbucket as well as Github and not have that reflected in their streak at all.

__My theory is that streaks, though well-intended, served no purpose.__

They were ripe for abuse and not a very good "reward" for contributing code on the side, nor a good indicator of how grey a developer's beard is. Not all good developers make commits every day, and encouraging a commit-a-day is getting people trapped in the Silicon Valley bad-habit of never not working. For the time being, good riddance.

At Canonical, we use Launchpad to host most of our source code. Launchpad has this concept of "Karma" which tracks your commits, code reviews, bug reports, branch merges, etc on Launchpad. Karma decays over time but not immediately, which means that you can take days off and maintain Karma. From this number, you can usually tell if a user is new to the system, highly active recently, or has turned into a manager. For reference, mine is [a little over 3000](https://launchpad.net/~larryprice/+karma) as of this blog post. Although I mostly like the way Karma works, I don't think it's the epitome of source code gamification.

As far as I know, sites like Bitbucket and GitLab don't have such a concept of rewarding heavy coders or giving some indication of recent activity outside of a news feed. Maybe a news feed style is enough?

The good people at Github want you to code socially, and I'm excited to see if they can come up with a fun and light way to gamify open-source software while avoiding some of the unfun aspects of streaks and the blaisé concept of news feeds.

What do you think? Do you miss streaks? Did streaks motivate you to contribute more frequently? Any ideas for a new system? Tell me in the comments!
