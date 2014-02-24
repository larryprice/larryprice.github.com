---
layout: post
title: "Letting the Browser Decide Betwen HTTP and HTTPS"
date: 2014-02-13 17:55:00 -0500
comments: true
categories: web
---

While working on SEP Labs' [Health2Wealth](//h2w.cc) app, I got my first taste of setting up a website to use secure connections. I've been using the browser extension [HTTPS Everywhere](//www.eff.org/https-everywhere) for quite some time now, so all of the web pages that I visit attempt to using `HTTPS`instead of the standard `HTTP`. When I inadvertently started testing our app with `HTTPS`, things got a little weird.

My machine was able to open most of a page, but it wouldn't load some of the graphics or javascript we had embedded in the page. I was the only one able to reproduce it, so we chalked it up to a wonky machine. Then I suddenly remembered that I was using HTTPS Everywhere. I found the area in FireFox that warns you about insecure connections and found that our page was only _partially_ loading on my browser because some of the content was insecure. From there I enabled an option to reveal the insecure elements, which forced my page to load everything correctly. So now we needed to find what was insecure about our page. I trekked through the codebase and found that some of our graphs and links were hard-coded to use `HTTP`. My initial thought was just to force them to use `HTTPS`, which is [almost certainly the impending standard](//lists.w3.org/Archives/Public/ietf-http-wg/2013OctDec/0625.html). With a quick search, I found that there exists a __real__ solution to this problem.

Let's say I have a very normal web application, so I want to include [jQuery](//jquery.com) embedded in my page. I initially copy-paste `<script src='http://code.jquery.com/jquery.js'></script>` right into my `<head>` node, which normally seems fine. Unfortunately, this is where issues arise. I'm now forcing the user to make a connection which may or may not be secure to fetch data from an insecure page.

By removing the `http:` from the `http://`, the issue of deciding whether to fetch the file using `HTTP` or `HTTPS` is left to the web browser. So my script include becomes `<script src='//code.jquery.com/jquery.js'></script>`. This also works for links, images, and web fonts. In fact, every link included in this post uses the same principal, where I leave the protocol out of what becomes the `href` tag.
