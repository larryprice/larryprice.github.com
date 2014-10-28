---
layout: post
title: "To System Test or Not to System Test"
date: 2014-10-28 17:42:06 -0400
comments: true
categories: non-technical
---

System tests: a boon to product verification, but also time taken away from raw development. System-level testing means bringing up the whole system and running through a set of steps to verify that the system is working properly on all levels.

We have lots of system tests on our project, some that check for the existence of UI elements, and some that verify that a user can go through the entire e-commerce process. Some of the tests require network access, so they take up to 3 or 4 minutes. I had gotten used to writing system tests with [ruby](https://www.ruby-lang.org/en/) and [capybara](https://github.com/jnicklas/capybara), which I've always found straightforward and easy to write. But this current project is in ASP.NET MVC, which seems to have greatly complicated bringing up and automatically manipulating the system. Our current test suite takes over an hour to run and results in at least 10 consistently-failing tests (referred to as "flaky" for dignity's sake).

Because of this, there has been talk about getting rid of the system tests.

Getting rid of system tests? Doesn't this go against everything we believe in? Think of TDD! Think of the children! Think of acceptance criteria! Won't somebody think of the children!

So I've been thinking: what is the purpose of system tests? The first thing that comes to mind is the obvious: to test the full stack. Often components work well individually but throw tantrums when forced to interact with the rest of the system.

Another obvious answer: system health. If the system tests are passing, then surely the site still works after my push! You did add a system test for that changeset, right?

I'm not satisfied with those answers. Those are great _side effects_ to writing system tests, but they aren't _sustainable_ reasons. For their own devious reasons, someone can always delete a system test that would have broken with your code change. You can always write your system test steps poorly, resulting in incorrect (though passing) behavior.

The best reason for writing system tests is __to find out what you've accomplished__.

Sometimes when I finish coding a new feature, I ask myself "MY GOD WHAT HAVE I DONE." There are unit tests in place, and they have great names like `SetsIsValidToTrue` and `ReturnsSomethingThatsNotNull` and `DoesntCrashTheSystem`, but they fail to tell me what just happened. So I switch context to the system tests. In writing a system test, I bring up the whole system, run through my Happy Path™ and my edge cases, and __I can see precisely what I've done__. I can watch my work succeed or fail. I can see when it takes way too many complicated page clicks to perform a task. I can see that when I click something too quickly the page fails to load. I can see that the URL is just plain _weird_. Most importantly, I can sit back and __see__ it happening.

So I push my system test to the build server and it works fine until a few dozen other changesets go through on the same page and eventually the system test becomes flaky.

Where did we go wrong?

I have an idea on how to deal with this, but you're probably not going to like it.

As soon as I push a system test, it becomes legacy. You are no longer allowed to edit this system test. If you are updating that feature, you are responsible to _delete_ and potentially _rewrite_ each relevant test. In doing this, you are now required to think about the way the whole system fits together every time you push a feature. System tests will become a picture frozen in time of the way the feature worked the last time it was worked on. System tests will no longer be _a rolling history of what the system did_, but _an exact specification of what the system does_. They become a piece of the development process to demonstrate the intended functionality of the feature.

In following this practice, we technically lose out on reusable system test code. I say good riddance, as testing code is the quickest part of any system to become bloated and convoluted. The loss in test creation speed could be worth the gain in developer knowledge of the intricacies of the system.
