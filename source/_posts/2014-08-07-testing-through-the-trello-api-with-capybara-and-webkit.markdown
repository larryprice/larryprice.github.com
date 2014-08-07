---
layout: post
title: "Testing through a Trello connection with Capybara and Webkit"
date: 2014-08-07 07:25:11 -0400
comments: true
categories: trello ollert capybara ruby sinatra
---

During the hardening of [Ollert](https://ollertapp.com), a Trello data analysis tool I wrote, I started writing acceptance tests. I quickly ran into an issue where the meat of my application requires opening pop-up window, signing into Trello, and allowing my application access.

I created a test user on Trello with a few varied boards to allow for proper testing. In doing this, I store the user's login information in my .env file. For the most part, I can use the steps provided in [this common web_steps.rb](https://gist.github.com/larryprice/546d6c029bb3074bd84c).

``` cucumber Connecting.feature
Feature: Landing

Background:
  Given I am on the landing page

@javascript
Scenario: Deny connecting to Trello
  Given I follow "Connect to Get Started"
  And I press "Deny" on the Trello popup
  Then I should be on the landing page

@javascript
Scenario: Allow connecting to Trello
  Given I follow "Connect to Get Started"
  When I authorize with Trello as the test user
  Then I should not see "Connecting..."
  And I should not see "Redirecting..."
  And I should be on the boards page
```

When the Trello popup appears, we have to specify the window we're going to use. Since I'm using [capybara-webkit](https://github.com/thoughtbot/capybara-webkit), I'm going to go ahead and do all of my Trello popup activities in one step, which saves me from writing a lot of unnecessary steps.

``` ruby trello_popup_steps.rb
When /^I press "(.*?)" on the Trello popup$/ do |button|
  trello_popup = windows.last
  page.within_window trello_popup do
    click_button button
  end
end

When /^I authorize with Trello as the test user$/ do
  trello_popup = windows.last
  page.within_window trello_popup do
    click_link "Log in"

    fill_in "user", with: ENV['TEST_USER_TRELLO_USERNAME']
    fill_in "password", with: ENV['TEST_USER_TRELLO_PASSWORD']
    
    click_button "Log In"
    click_button "Allow"
  end
end
```

Straightforward so far. We grab the window handle and we click links, fill in fields, and press buttons within that window.

Note that I'm using capybara-webkit, a headless web driver, to run my Javascript. Although the first test ("Deny") will pass, the "Allow" test fails ambiguously. This is because capybara-webkit is not recognized as a supported browser by the Trello popup.

Anecdotally, I contacted Trello support about this and received the following response:

> Currently it is not possible to test this with a headless browser as you are looking to do without getting the unsupported browser message.

So I guess we should just give up, right? ...Or we could manipulate the headers we send to load the Trello popup such that Trello _thinks_ we are Google Chromium.

``` cucumber trello_popup_steps.rb
When /^I authorize with Trello as the test user$/ do
  trello_popup = windows.last
  page.within_window trello_popup do
    page.driver.header(
      "User-Agent",
      "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/34.0.1847.116 Chrome/34.0.1847.116 Safari/537.36"
    )

    click_link "Log in"

    fill_in "user", with: ENV['TEST_USER_TRELLO_USERNAME']
    fill_in "password", with: ENV['TEST_USER_TRELLO_PASSWORD']
    
    click_button "Log In"
    click_button "Allow"
  end
end
```

Fantastic. Now my tests pass. I can't sleep at night, but my tests pass.

Unfortunately, that won't be the case if I add more tests to this `.feature` file. Hidden somewhere deep in the browser's cache or cookies or somethings, Trello is remembering that we logged in sometimes. Sometimes it even remembers that someone else has logged in. The UI of the Trello popup changes based on whether it thinks you've already logged in. In order to keep things consistent, I like to add an if-statement to take care of this case.

``` cucumber trello_popup_steps.rb
When /^I authorize with Trello as the test user$/ do
  trello_popup = windows.last
  page.within_window trello_popup do
    page.driver.header(
      "User-Agent",
      "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/34.0.1847.116 Chrome/34.0.1847.116 Safari/537.36"
    )

    if page.has_content? "Switch Accounts"
      click_link "Switch Accounts"
    else
      click_link "Log in"
    end

    fill_in "user", with: ENV['TEST_USER_TRELLO_USERNAME']
    fill_in "password", with: ENV['TEST_USER_TRELLO_PASSWORD']
    
    click_button "Log In"
    click_button "Allow"
  end
end
```

Edge cases addressed. Now I can make connections to Trello and test my application. Be warned, I've already had these tests break once when Trello updated the UI behind the Trello popup. If Trello ever stops supporting Chromium 34.0, these tests are also likely to stop working. These tests are most useful during development, when we have the potential to break the Trello connection ourselves, and so I think they are well worth the pain of potential future maintenance.
