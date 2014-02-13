---
layout: post
title: "Custom Domain with Github Pages"
date: 2014-02-12 21:30:00 -0500
comments: true
categories: web
---

As of today, I'm the proud owner of [larry-price.com](larry-price.com). On the recommendation from a friend, I used [domainmonster.com](domainmonster.com) for it's cost effectiveness and usable toolset. I wanted to link my Github Pages blog (this blog!) to my new site. In case I ever want to do this again, I've provided myself (and you!) with a step-by-step guide below. Note that Github [has a page that explains how to do this](https://help.github.com/articles/setting-up-a-custom-domain-with-pages), but there's some fluff and ordering issues that turned me off from using those instructions as any more than a reference.

####Step 0: Buy your domain####

All sites where you can purchase domains have the same domains available, but at different prices. From the sites I perused, [GoDaddy](godaddy.com) had the best search for domain names, suggesting different variations on the searched domain name. I found that [domainmonster.com](domainmonster.com) had the ugliest UI but the best prices. [Hover.com](hover.com) looked really modern and had moderate prices. Shop around and spend money.

####Step 0.5: Exercise restraint####

At this point in the process, __don't go to your site__.

My guess is you already have. If you don't have the ability to go back in time, don't sweat it, just know that you probably won't be able to verify the fruits of your labor for a few hours.

####Step 1: Setting up an 'A Record'####

Find the ip address where your site is currently hosted. For me (replace URL with your own):

``` bash
$ nslookup larryprice.github.io
Server:   127.0.1.1
Address:  127.0.1.1#53

Non-authoritative answer:
Name: larryprice.github.io
Address: 199.27.74.133
```

You should be able to "manage" your domain on your domain provider's website. In the "management" area, you should be able to find a section about "DNS". Find something labeled "A records." There will likely already be one or two entries here - one for the IP address of your new domain and one with the IP address of your domain with "www." prepended.

Modify the entry (or entries) with the IP address found using `nslookup` ("199.27.74.133" from the example above). Save your site settings.

####Step 2: Mucking with the "www" instance####

While we're modifying these settings, we might as well move the 'www' case out of the "A records". Go ahead and delete it.

Look for something named "CNAME records", or figure out how to add records in general. We're going to add a CNAME record where the "Alias" is "www" and the "Address" is your site's name. In my case, my "Alias" field is "www" and my "Address" field is "larry-price.com". Add the record and cross your fingers for success.

####Step 3: Telling Github Pages what to do####

Alhtough Github Pages is magical in its own right, it doesn't know what to do when receiving a request from the "A record" we created in Step 2 without further instruction.

If you're using Github Pages for a user page, we are going to make this change on the `master` branch. If this is a project page, do it on the `gh-pages` branch.

All we have to do is put our new domain into a file in the root folder called `CNAME`. Let's do it in one fell swoop of the command line.

``` bash
$ echo "larry-price.com" > CNAME 
```

Push that to Github and wait 0 to 10 minutes for Github to refresh your page.

Note: If you're one of the few, the proud using the brilliant [Octopress](http://octopress.org/) to generate your blog, you want to make sure to add the `CNAME` file to the root of your `source/` directory. That way, when you call `deploy` your `CNAME` file will end up in the root of your `_deploy` directory.

####Step 4: Behold####

Check your Github repository's "Settings" section. In the "GitHub Pages" section, you should now see something along the lines of "Your site is published at http://larry-price.com". Go, young one, and visit your new site; and behold the glory you have brought to your family.

I'll note that this is the point where not obeying Step 0.5 will cause issues. Once your internets knows that your page hit a certain DNS, it takes a few hours to refresh. In my case it only took about 2 hours, but it may take between 12 and 48 hours to refresh. A workaround to see if everything is working is to pull out your smart phone and use your carrier's network to check your new site, since that area of the internets is probably still untainted. (All of this paragraph is a simplified thought process on how DNS works, considering that's the level I understand it.)

Now fly, my children, and spread yourself over the internet with dozens of silly domain names!
