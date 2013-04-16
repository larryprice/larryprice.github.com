---
layout: post
title: "Tags in C++ Cucumber tests"
date: 2013-04-15 22:10
comments: true
categories: cucumber c++ testing
---

The project I'm working on is slowly adding [Cucumber](https://github.com/cucumber/cucumber-cpp) acceptance tests to our massive code base in order to replace aging [Robot tests](https://code.google.com/p/robotframework/). One would think that getting developers on our team to use Cucumber would be east, since it uses [googletest](https://code.google.com/p/googletest/) and [googlemock](https://code.google.com/p/googlemock/) which we use for our unit tests. Unfortunately, very few people on the team have been motivated to write integration tests using the new framework, so I have very few people to go to when I have problems writing my own integration tests.

The area of the code I deal with uses [embedded mono](http://www.mono-project.com/Embedding_Mono) to communicate with some C# libraries that we share with other applications. This means we have unmanaged memory which talks with managed memory. This has caused us more headaches than I care to remember. One such problem is that we have a static object that we only want to create and destroy once. So I write my first Cucumber test:

``` cucumber DoStuff.feature
Feature: Do that thing that we have to do

Scenario: Do it my way
  Given I have done step 1
  When I do step 2
  Then I should see results
```

``` c++ DoStuff_StepDefinitions.cpp
#include <cucumber-cpp/defs.hpp>
#include <gtest/gtest.h>
#include <mono/jit/jit.h>

const QString DOMAIN_NAME = "bridge";

class Context
{
  // The static Mono object
  static MonoDomain *Domain;
}

BEFORE() { Context::Domain = mono_jit_init(DOMAIN_NAME); }

AFTER() { mono_jit_cleanup(Context::Domain); }

GIVEN("^I have done step 1$") { /* ... */ }

WHEN("^I do step 2$") { /* ... */ }

THEN("^I should see results$") { /* ... */ }

```

Before my scenario starts, the ```BEFORE()``` function is called and my MonoDomain object is created. When the scenario ends, my ```AFTER()``` statement is called and the objects in my MonoDomain are cleaned up. Now, I add a second scenario.

``` cucumber DoStuff.feature
Feature: Do that thing that we have to do

Scenario: Do it my way
  ...

Scenario: Do it your way
  ...
```

Now I run my Cucumber test, and Mono explodes. Why? Because the ```BEFORE()``` and ```AFTER()``` functions are not _before all_ and _after all_, but _before each_ and _after each_.

So what should we do? Move the function calls in the ```BEFORE()``` and ```AFTER()``` statements into the constructor and destructor of the Context class?

Same problem. Are there ```BEFORE_ALL()``` and ```AFTER_ALL()``` macros? No.

I began to panic. I asked the person who taught me how to write Cucumber tests in C++. Our idea was to create the MonoDomain during what I knew would be the first step, and delete it after what I knew would be the last step. Oh, the horror! That would mean not being able to reuse those steps, not to mention moving the creation/destruction code around anytime I wanted to add new steps or change the order of my previous steps. We also thought about making specific steps and sticking them at the front of the first scenario and at the end of the last scenario. This still meant that the lay developer would have to recognize these first and last steps from the others. I asked my local senior engineer, and his advice was to create separate Cucumber tests for each scenario I intended to create. My plan was to write 6 scenarios in the long-term for this feature, and I really didn't want to turn these very similar tests with beautifully reusable steps into 6 features.

Then it hit me: Cucumber is open source. I found the source [on Github](https://github.com/cucumber/cucumber-cpp) and started looking through [the example code](https://github.com/cucumber/cucumber-cpp/tree/master/examples/). It was there that I discovered [tags](https://github.com/cucumber/cucumber-cpp/tree/master/examples/FeatureShowcase/tag). Tags were the solution to my problem.

``` cucumber DoStuff.feature
Feature: Do that thing that we have to do

@first
Scenario: Do it my way
  ...

@last
Scenario: Do it your way
  ...
```

Using tags, I could label my scenarios with meaningful ```@first``` and ```@last``` tags to signify the beginning and end of my tests. The trick is to then add the required tags to my ```BEFORE()``` and ```AFTER()``` macro as such:

``` c++ DoStuff_StepDefinitions.cpp
#include <cucumber-cpp/defs.hpp>
#include <gtest/gtest.h>
#include <mono/jit/jit.h>

const QString DOMAIN_NAME = "bridge";

class Context
{
  // The static Mono object
  static MonoDomain *Domain;
}

BEFORE("@first") { Context::Domain = mono_jit_init(DOMAIN_NAME); }

AFTER("@last") { mono_jit_cleanup(Context::Domain); }

// ...

```

Now my MonoDomain is only created _before_ the scenario labeled ```@first``` and _after_ the scenario labeled ```@last```. Obviously, this isn't the cleanest fix imaginable, but it was the cleanest fix _available_. Whenever someone wants to add a new step to this test, they need to remember to move the ```@last``` tag to their scenario. However, I have the hope that it will be pretty obvious that the second scenario is no longer "last" when there is a third scenario following the "last" scenario. Anyway, it leaves me happy enough, since now my tests don't explode and I'm able to reuse ~50% of the steps I had already written for the first scenario. I added a third scenario later on and 9 out of the 10 steps in the scenario were reused from the first and second scenario.

There are lots of other cool things you can do with Cucumber tags, like having multiple tags on objects. All tags that match ```@first``` will do one thing, but tags that match ```@first``` and ```@second``` can have multiple ```BEFORE()``` or ```AFTER()``` clauses.
