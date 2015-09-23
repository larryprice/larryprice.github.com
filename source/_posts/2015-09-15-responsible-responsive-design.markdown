---
layout: post
title: "Responsible Responsive Design"
date: 2015-09-15 22:05:57 -0400
comments: true
categories: write-ups
---

### The Gist ###

In this modern age of infinite internet-capable devices, web developers have found a "silver bullet" which we call responsive design. Unfortunately, responsive design is often coupled with large CSS files, delayed element styling, and an overabundance of javascript. Scott Jehl has penned [Responsible Responsive Design](//abookapart.com/products/responsible-responsive-design) to help us make the web better. The strategies outlined in this book will make your web pages smarter, faster, and lighter.

### Takeaways ###

We are spoiled by 4G, unlimited data, fiber, and the latest smart devices. Most of the world does not have the same luxury as the majority of web designers. We severely limit our userbase when we consider our own lifestyles as the norm. If you want any chance of obtaining a global audience, your site _must_ be able to deal with slow connections, data limits, small screens, and outdated browsers.

The average site still takes 10 seconds to load. Despite generally better network speeds, we have started delivering more CSS, javascript and markup; bigger, higher res images when they aren't always necessary; and long-running javascript to take advantage of the latest and greatest framework. We can and should work to deliver only the content users need.

We can load the top part of the page a user will see before loading the rest of the page by pulling some of our styles into the `<head>`. In doing this, we can prevent a Flash of Unstyled Content (FOUC) when the user first visits the page. There are tools to do this mentioned in the book, such as [criticalcss](https://github.com/filamentgroup/criticalcss). You can go a step further and load the rest of your CSS at the end of the event queue. Even better: only deliver that experience the first time the user visits that page, after which the browser will have cached the CSS and you can load it through the usual means.

Design your first drafts as naked HTML and CSS. This will be the base experience for a user having a really bad day. Users using old technology, block or fail to load javascript, or experience a bad connection will be able to experience a decent version of your site. Iterate to add a better experience for users who can load javascript. Iterate again to add HTML5 or CSS3 features which may not be available in all browsers. The important aspect is that the app will always degrade gracefully.

Start with the mobile version. If you can design for a small screen with weird gesture inputs, you should be able to translate that to a tablet and desktop machine. Use `@media` magic to deliver the right content.

Use HTML5 elements when possible. `<picture>` and `<video>` can be very efficient in choosing which images or videos to load from the server, preventing the client from downloading the `1x`, `2x`, `3x` versions of your images. Don't forget about `<svg>` either for handling vector graphics.

Screen readers read _everything visible_. This can cause a lot of bad things to happen. Using `display: none` will prevent screen readers from telling the user about your collapsed items (think of all the wasted hamburgers). Elements strategically hidden with `z-index` are listed by the screen reader as a visible element, potentially misinterpreting your intent. Markup created dynamically could have a drastic effect on that user's experience. We should be diligent with our markup to remain inclusive.

Polyfill, shims, and shivs are some common terms to refer to libraries which fake modern browser functionality into older browsers. `html5shiv`, `selectivizr`, and `modernizr` can be described by these words.

### My Action Items ###

* __Start Small.__ Although I've had several projects where I/we start out saying we'll be mobile first, I don't believe this has ever actually happened to me. Next time I start a web project I want to start with mobile and expand.
* __Degrade Gracefully.__ I'm notably terrible at this. I don't think I've ever created a site which will respond well with javascript disabled let alone any HTML5 features which may be missing. I always design my websites with my own devices and connection speeds in mind. For the time being, I will also put this item off until I have the chance to start a project from scratch.
* __Pay Attention to Accessibility.__ Accessibility hasn't been on my mind much when throwing together web sites. If I get some time, I'd like to look into adding better screen reader attributes into a live web application like [Ollert](//ollertapp.com).
* __Page load budgets.__ Jehl discusses getting strict about page load time. Keep it low and weigh load time effects heavily when deciding to add new features to a site. When I add new features to pages, I intend to keep this in mind. Look for this to affect something like [Ollert](//ollertapp.com) if I ever get around to redesigning the dashboard.
