---
layout: post
title: "Lessons Learned during the Conception of Ollert"
date: 2014-06-12 20:32:00 -0400
comments: true
categories: non-technical ollert
---

During [SEP's](http://sep.com) previous [startup weekend](http://sep.com/labs/), I pitched an idea for a [Trello](http://trello.com) Analysis Tool called [Ollert](https://ollertapp.com). In less than three days, a team of 6 built a minimal viable product (MVP) and put it live on the internet. In a little over three months, I have slowly guided Ollert through the legal department, obtained a real domain with security, and fixed a few bugs along the way. I've learned a thing or two about seeing a hackathon project to fruition that I'd like to get in writing.

###1. You Can Never Be Too Prepared

Before Startup Weekend, I spent days refining my idea and coming up with a cute little proof of concept. I even laid out work items for my developers to work on. When I finally discovered a clever name, I thought my preparation was over.

I was wrong.

Although I requested 5 developers to help me build Ollert, my plans only allowed for two developers to work simultaneously. I didn't realize how hard it would be to keep everyone busy all the time, especially my less-experienced engineers.

Although I did proof-of-concept my idea before we started, I failed to proof-of-concept the more dynamic capabilities a multi-user web application needs to provide. This mistake cost another developer and I the first night of the weekend, plus a bit of the next morning.

Don't even get me started about environment setup. Although I run [Ubuntu](http://www.ubuntu.com) natively and had all my developer tools ready before we started work Friday night, the rest of my team were users of That Other Operating System. I should have created a virtual machine with everything all set up, burned it to several USB drives, and let my developers set up VirtualBox without my intervention.

###2. Talk to Legal

Why do I care about legal? Isn't this my responsibility anyway?

As it turns out, putting a startup weekend project live on the internet before the weekend is over is really cool from a team perspective, but terrifying from a legal perspective. With our MVP, I wasn't using SSL to encrypt user data. I didn't consider how Fog Creek would react once they realized our name is their name backwards. I even had the chutzpah to stick my company's logo in my site's footer.

A quick discussion with management will prepare both sides for the "grand reveal" if the project makes it to launch. Personally, I'd like to see some level of legal counsel in the Startup Weekend "pre-pitch" session to get feedback before implementation.

###3. Listen

The second day, one of my developers mentioned using endpoints in the application, and I rejected this idea on the basis that it was too complicated. On the dawn of the final day, we realized that we needed to use his approach or the application would be unusable. This required us to do a lot of rework that could have been avoided. I often think about what other ideas might have been suggested by my team that I may have accidentally ignored.

###4. Easy Tasks

My assumption going into Startup Weekend was that my team would all be familiar enough with the technology to be able to "jump right in" or follow along with someone who could. This was a bad assumption.

It's easy to find tasks that _I_ can do. It's much more difficult to find tasks that _anyone_ could do. What are the less-involved tasks on your current project that a less-experienced developer could work on until they're ready to tackle something bigger? Can they set up the database, or the tests? Can you find a guide for them to follow to do these things? If not, you'll have people on your team who don't feel involved but desperately want to help.

###5. Start Small

Ollert is a pretty big idea, especially for a three-day project. I had high hopes of making it even bigger until the dawn of the final day. I wanted to include some sort of payment system to prove that we would be able to charge people when they sign up without actually charging them. This "feature" of fake payment has no place in a real product, and would have been misleading at best. We had dozens of ideas for statistics and charts to make it into the website that just weren't that useful.

Limiting scope might have allowed us to come up with a more polished MVP; I walked away the final evening wishing I had left out sign up/in in favor of giving the application more sex appeal.

###Conclusion

Starting a project is hard. It's even more difficult when you have to delegate most of the work to other people. Startup Weekend is a manger and a mortuary, seeing the birth of many ideas and the death of most. I hope to take this experience and make an even better new product next Startup Weekend.
