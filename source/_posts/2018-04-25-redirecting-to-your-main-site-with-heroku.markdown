---
layout: post
title: "Redirecting To Your Main Site With Heroku"
date: 2018-04-25 20:52:19 -0500
comments: true
categories: php heroku web
---

We have a lot of domains that we want to redirect to the same server, but we use a DNS service that does not allow doing a domain forward, and we're not allowed to upgrade. I wanted to do this in the simplest way possible, so I created a workaround using a PHP script and Heroku. The source discussed in detail in this post is available on GitHub: [https://github.com/larryprice/simple-heroku-redirect-app](https://github.com/larryprice/simple-heroku-redirect-app).

The goal here is for users to visit a page and then be immediately redirected to the new site. I've defined two environment variables to be used in this project: `SITENAME`, a human-readable name for our website, and `SITEURL`, the full URL that we actually want the user to end up on. I've defined a PHP file called `index.php`:

``` php index.php
<!DOCTYPE html>
<html>
  <head>
    <title><?php echo getenv('SITENAME') ?> - You will be redirected shortly...</title>
    <meta http-equiv="refresh" content="0;URL='<?php echo getenv('SITEURL') ?>'" />
  </head>
  <body>
    <p>Please visit the official <?php echo getenv('SITENAME') ?> site at <a href="<?php echo getenv('SITEURL') ?>"><?php echo getenv('SITEURL') ?></a>.</p>
  </body>
</html>
```

The important piece here is the `<meta>` tag, which actually does the redirect for us. The only PHP code here are `echo getenv` commands that render our environment variables in the template. Since I'm a PHP novice, there may be a better way to do this, but the `echo` works just fine.

We also need to tell Apache how to serve the application. We want to match any routes and render our `index.php`. So we create an `.htcaccess` file:

``` sh .htaccess
RewriteEngine on
RewriteRule . index.php [L]
```

To satisfy Heroku, we need to list the dependencies for our PHP application. Fortunately for us, we don't have any dependencies that Heroku does not provide by default. We'll just create a `composer.json` file in the root of our project with an empty object:

``` json composer.json
{}
```

That's everything we need. You could recreate the project, but you could also just pull down the project listed above and push it up to Heroku:

``` bash
$ git clone https://github.com/larryprice/simple-heroku-redirect-app.git
$ cd simple-heroku-redirect-app
$ heroku create
$ git push heroku master
```

With your application available on Heroku, we still need to set the environment variables described earlier as [config variables](https://devcenter.heroku.com/articles/config-vars):

``` bash
$ heroku config:set SITENAME=yourgooddomain.com
$ heroku config:set "SITEURL=Your Good Domain's Website Name"
```

Now tell Heroku all [the domains](https://devcenter.heroku.com/articles/custom-domains) that will be accessing this application. These are the domains you want users _not_ to use:

``` bash
$ heroku domains:add yourbaddomain.com
$ heroku domains:add www.yourbaddomain.com
```

Now you just need to add the records indicated by the above command to your DNS records. These will probably be CNAME records pointing from `@` to `yourbaddomain.com.herokudns.com` or `www` to `yourbaddomain.com.herokudns.com`.
