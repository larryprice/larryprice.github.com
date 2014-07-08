---
layout: post
title: "Sending emails with Pony and Sendgrid"
date: 2014-07-08 05:53:26 -0400
comments: true
categories: ruby web sinatra pony heroku sendgrid
---

It's incredible how easy it is to send emails through a web application, there's no wonder we get so much spam. Assuming we have a [ruby](http://ruby-lang.org) app using [Sinatra](http://sinatrarb.com), [Pony](http://adam.herokuapp.com/past/2008/11/2/pony_the_express_way_to_send_email_from_ruby/) is one of the easiest ways to get started with your own spam empire.

Installation is, as always, trivial with ruby. Install the gem with `gem install pony` or add `gem pony` to your `Gemfile` and run `bundle install`.

I like to configure Pony in my application's `configure` block. I could also add it to my `config.ru`, but I like to keep that file as tiny as posible to avoid having configuration code all over the place.

``` ruby web.rb
# ...

require 'pony'
require 'sinatra/base'

class Application < Sinatra::Base
  configure do
    # ...

    Pony.options = {
      :via => :smtp,
      :via_options => {
        :address => 'smtp.sendgrid.net',
        :port => '587',
        :domain => 'myapp.com',
        :user_name => ENV['SENDGRID_USERNAME'],
        :password => ENV['SENDGRID_PASSWORD'],
        :authentication => :plain,
        :enable_starttls_auto => true
      }
    }
  end

  # ...
end
```

This block tells Pony to use the [SendGrid](http://sendgrid.com/) server to send mail, use the "myapp.com" HELO domain, and dig up the username and password fields from my environment.

If you're using [Heroku](https://heroku.com) to host your application, you can [sign up for a SendGrid account through your Heroku app](https://addons.heroku.com/sendgrid), which gives you instant access to your SendGrid account. The `username` and `password` field you need to fill in your environment are automatically populated in your Heroku config, which you can view by running `heroku config` for your application. The free account gets you up to 200 emails a day.

Since I might have multiple developers working in my source code and testing the email-sending functionality, I have all the developers [sign up for their own free SendGrid account](https://sendgrid.com/user/signup). This should help to alleviate some of the email volume from any particular account while developing. After signing up, it took my account nearly 4 hours to be "provisioned" (see: approved) by the SendGrid team. Once you're approved you can start sending emails using your developer account credentials. I stick my username/password in my local `.env` file (another reason to make sure you're not storing your environment on your server or in your git repo).

So let's actually send an email. Let's create a route that sends an email to verify a new user account; I'll take some liberties by saying we have a `User` model defined already that generates a signup verification hash. I can tell pony to send a plaintext body through the `body` option and an HTML body through the `html_body` option.

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
```

``` haml views/verify_account_email.rb
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

When you have a user hit this route, an email will be sent to the user with the given subject, to, from, and body fields using the configuration parameters given in the previous `configure` block. Fast, easy, and, best of all, no `sendmail` configuration.
