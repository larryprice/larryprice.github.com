---
layout: post
title: "Testing automated emails with email-spec in Cucumber"
date: 2014-07-12 07:36:33 -0400
comments: true
categories: web ruby sinatra ollert
---

Now that [I send emails using Pony](/blog/2014/07/07/deauthorizing-token-with-the-trello-client/), I want to be able to verify that the emails are being generated correctly. I also don't want to send real emails and have my tests check an inbox somewhere. I found a couple of solutions to do this, including [pony-test](https://github.com/johnmendonca/pony-test) and [email-spec](https://github.com/bmabey/email-spec). Although pony-test fits my needs perfectly, the last commit was December 27, 2011 (2.5 years ago at the time of this post), and thus was using an outdated version of [capybara](https://github.com/jnicklas/capybara) which I was unwilling to use. Fortunately, pony-spec is mostly just a fork of email-spec with all the non-Pony components ripped out.

I'm going to be using Cucumber to test my emails, but email-spec also boasts compatibility with rspec and Turnip. To get started:

``` bash
$ gem install email-spec
```

The developers of email-spec were kind enough to give us some free step definitions. If I was using rails, I could just type `rails generate email_spec:steps`, but since I'm using Sinatra I opted just to copy-paste the file into my `step_definitions/` directory. You can find `email_steps.rb` [on Github](https://raw.githubusercontent.com/bmabey/email-spec/master/lib/generators/email_spec/steps/templates/email_steps.rb).

In [my last post about small horses and emails](/blog/2014/07/07/deauthorizing-token-with-the-trello-client/), I used the following code to send a confirmation email on signup:

``` ruby web.rb
# ...
post '/signup' do
  user = User.create! params

  url = "#{request.base_url}/account/reset/#{user.generate_verification_hash}"
  Pony.mail(
    to: user.email,
    from: "MyApp Help Desk <noreply@myapp.com>",
    subject: "MyApp Account Verification",
    body: "A request has been made to verify your MyApp account (https://myapp.com)." +
          "If you made this request, go to " + url + ". If you did not make this request, ignore this email.",
    html_body: haml(
      :verify_account_email,
      layout: false,
      locals: {
        email: user.email,
        date: DateTime.now.strftime("%H:%M:%S%P %B %d, %Y"),
        ip: request.ip,
        url: url
      }
    )
  )
end
# ...
```

``` haml /views/verify_account_email.haml
%p
  Hello!
%p
  An account verification has been requested for your new <a href="https://myapp.com">MyApp</a> account.

%ul
  %li
    Username: #{locals[:email]}
  %li
    Time: #{locals[:date]}
  %li
    IP address: #{locals[:ip]}

%p
  If you made this request, click the link below or copy-paste the following URL into your browser to verify your account:

%p
  %a{href: "#{locals[:url]}", alt: "Verify", title: "Click to verify account"}
    #{locals[:url]}

%p
  If you did not request this new account, please ignore this email.

%p
  Sincerely,
  %br
  Team MyApp

%p
  This email account is not monitored and will not receive replies. For more information, contact <a href="mailto:connect@myapp.com">connect@myapp.com</a>.
```

Given the pre-defined steps from email-spec, testing that this email gets sent is a breeze. Adding a scenario to my feature file:

``` cucumber features/SignupConfirmation.feature
Feature: Signup Confirmation
  As a new user
  When I sign up
  I should receive a confirmation email

Background:
  Given a clear email queue
  When I go to the signup page
  And I fill in "email" with "prez@whitehouse.gov"
  And I fill in "password" with "bunnies"
  And I press "Sign Up"
  Then "prez@whitehouse.gov" should receive an email

Scenario: Receives email with correct contents
  When "prez@whitehouse.gov" opens the email
  Then they should see the email delivered from "MyApp Help Desk <noreply@myapp.com>"
  And they should see "MyApp Account Verification" in the email subject
  And they should see "Username: prez@whitehouse.gov" in the email body
  And they should see "An account verification has been requested"
```

That's it. Now we know that an email like the one above will be sent during signup. What we can't test here is that our SMTP server (or equivalent) is working, so in reality I'm only testing that the email will attempt to send that looks like the one I test against.
