---
layout: post
title: "Hooks in cucumber-cpp"
date: 2013-11-22 20:40
comments: true
categories: cucumber c++ testing foss
---

A few months ago [I blogged about tags](/blog/2013/04/15/tags-in-c-plus-plus-cucumber-tests/) in [cucumber-c++](https://github.com/cucumber/cucumber-cpp). The scenario I presented involved using tags to call a `BEFORE` hook before the first scenario and an `AFTER` hook after the last scenario. The code looked a little bit like this:

``` cucumber DoStuff.feature
@first
Scenario: Do it my way
  ...

Scenario: Don't do it
  ...

@last
Scenario: Do it your way
  ...
```

``` c++ DoStuff_StepDefinitions.cpp

BEFORE("@first") { cout << "This is the first step!" << endl; }

AFTER("@last") { cout << "This is the last step!" << endl; }

...

```

The scnenario labeled `@first` would call the corresponding `BEFORE` macro and the `@last` scenario would call the `AFTER` macro. If I didn't have tags in place, the macros would have both been invoked before/after each scenario. Macros for `BEFORE_STEP` and `AROUND_STEP` are also available; `BEFORE_STEP` allows you to tag individual steps and `AROUND_STEP` acts as a before/after for individual steps.

This was a workaround. What I really wanted to do was to not use tags, and instead unconditionally perform an action before the first scenario was run and after the last scenario is complete. Since cucumber-cpp is open source, I decided to implement that a few weeks ago ([see changeset](https://github.com/cucumber/cucumber-cpp/commit/26e11d0248edf32a8bac17df9d2d4ceb135ed502)). Now the above example becomes:

``` cucumber DoStuff.feature

Scenario: Do it my way
  ...

Scenario: Don't do it
  ...

Scenario: Do it your way
  ...
```

``` c++ DoStuff_StepDefinitions.cpp

BEFORE_ALL() { cout << "This is the first step!" << endl; }

AFTER_ALL() { cout << "This is the last step!" << endl; }

...

```

This is the same behavior as the first example, except I don't have to force my fellow developers to move tags around when they add/remove scenarios. Also now my fellow developers can stop asking me why I was using tags and why when they added a scenario they couldn't get the tests to pass.

The moral of the story is that you should go implement that feature you want to see in your favorite open source project.
