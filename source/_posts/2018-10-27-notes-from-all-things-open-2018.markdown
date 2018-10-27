---
layout: post
title: "Notes from All Things Open 2018"
date: 2018-10-27 15:56:45 -0500
comments: true
categories:
---

This is a brief overview of the talks I sat through while attending [All Things Open 2018](https://allthingsopen.org/) in Raleigh, NC.

### Day 1

First day was okay, but I had trouble finding sessions that interested me and weren't geared towards introductory use.

#### Keynotes

The first keynote, "The Next Billion Internet Users" by Angela Oduor Lungati, described the rapid rise in internet users in Africa and Asia. Her team made their app mobile-first, as many users only have access to the internet on a smart device. This allowed the app to be used in many different and unexpected situations. Increased connectivity also allows more people to participate in the world of software. According to a recent GitHub survey, Asia is opening the largest number of repos on the site.

Burr Sutter of Red Hat talked about Istio, Red Hat's "service mesh" system. It's a pretty neat way to manage services with k8s and OpenShift. Users can launch multiple service containers with different features and seamlessly direct traffic to these containers based on certain rules. Users could even direct traffic to a new and old version of a container to determine how a new version interacts with a production environment, with end-users only ever interacting with the old version.

"The Next Big Wave" (Zaheda Bhorat) mostly focused on how to create a welcoming open-source project that's easy to contribute to, especially in a rapidly more connected world. As usual, READMEs and CONTRIBUTING docs are king, as well as good tutorials, wikis, and getting started guides.

In "Design in Open Source", Una Kravets discusssed how Design Thinking can benefit open-source projects. Unfortunately, it's really difficult to get designers to participate.

#### Track Sessions

"Turning 'Wat' into 'Why'" (Katie McLaughlin) brought up a few idiosyncrasies from many different languages and discussed _why_ the language behaves in that manner. No blame; just curiosity.

"Why Modern Apps Need a New Application Server" (Nick Shadrin) was an overview of the new Nginx Unit project, iterating on `nginx` with a focus on microservice architectures. This system actually launches applications, and several libraries/packages/modules are available for things like NodeJS and Go to enable this functionality. Configuration of any language was nearly identical, and defining the number of running instances was really easy through JSON endpoints. Auto-scaling was also included out-of-the-box.

"Open Data: Notes from the Field" was a panel discussion on how the Research Triangle uses citizens' data to make decisions. Much of the data used is decided upon on a municipal level as opposed to federal or state.

"Using Open Source for Large Scale Simulation in Robotic and Autonomous Driving Applications" (Nate Koenig) was largely a discussion about tools used to simulate robots. Obviously, testing robots in real life can be dangerous and expensive, so advanced simulation technology is crucial to iterating fast on this kind of hardware.

"React Already Did That" (Dylan Schiemann) hit on how React has evolved our ecosystem; components and functional programming will leave a permanent mark on JS development. Although React may not be around in 5 years, it is highly likely that the popular frameworks at that time will be fairly similar (think: Vue, Ionic, Svelte). This talk sort of devolved into a discussion of the speaker's "competing" technology Dojo, which was somewhat of a precursor to React. It also uses TypeScript, which reminds me a lot of the tech stack we use at Granular.

"You XSS Your Life! How do we keep failing at security on the web?" (David Rogers) was an overview of how easy it is to fall for cross-site scripting attacks in modern web applications. Malicious user input could take down our system or reveal user data, so we should be scrubbing data anywhere it gets entered. Lots of tools available. Although this is touched upon a lot, I know that I'm guilty of just taking user input and using it unthinkingly.

### Day 2

I found more relevant sessions to go to during the second day, which surprised me as normally the "last" day of a conference is worse than the first.

#### Keynotes

"Five Things You Didn't Know Python Can Do!" by Nina Zakharenko went over things I already knew Python could do. Python runs important code in all industries, and has found itself indispensable in the world of science.

Babel developer Henry Zhu gave a talk titled "Maintainer's Virtual" describing the world of full-time open-source development. Zhu left his job and works on Babel based on donations from the community. He talked about the guilt associated with taking breaks when people are donating their money to you, and how that easily leads to burnout. He talked about trying not to put too much pressure on yourself to be constantly contributing.

The final keynote, "Money as an Open Protocol" by Andreas Antonopoulos, was... interesting. A dash of conspiracy theory and anarchism made this talk a little uncomfortable. Big banks are not our friends, and this speaker was adamant that we would see the fall of centralized banking in the next 20 years. Bitcoin and friends are the predecessor to a new global digital currency. The choice we'll be facing soon is whether we have a decentralized open currency akin to Bitcoin as our primary form of money or something more insidious such as "Facebook Coin", "Google Coin", "Apple Coin", or "America Coin." A fun quote from this talk was "The opposite of authority is autonomy." Also "If money is the root of all evil, then `sudo evil`." Although this talk was captivating, it felt like a pitch for a dystopian novel. Crowd ate it up.

#### Track Sessions

Kyle Simpson's "Keep Betting on JavaScript" was probably my favorite session. Kyle gave a brief history of JavaScript, from its creation through its stagnation to the rapidly-evolving language we all know and love today. JavaScript's failures to change in the 00s was largely due to a lack of unity in the community, ultimately leading to a spec that was thrown away. Other languages began to appear that looked like they would leave JS in the dust. Just as JS was on the brink of death, the community united, new features were specced out, and JS rose from the ashes. Many people still hate on JavaScript, and this is largely due to the fact that they are "emotionally attached to the idea that JavaScript sucks." JavaScript is lingua franca in programming; it's readable by developers of many languages, and ideas can easily be expressed. Kyle was very much into progressive web applications, with native apps becoming an unnecessary part of the ecosystem. Every app should have at least one ServiceWorker to guarantee that a tab will continue to exist, even after we get on an airplane. "TypeScript is a really intelligent linter", Kyle says, but aside from that, can begin to confuse the world we live in if we use too many extended features. Transforming our code with all of these tools can make debugging harder, and can make it difficult for other developers to figure out what we've done using "View Source." "View Source" is the ultimate tool in a new developer's toolkit, allowing them to see how a site works and helping them develop new ideas. Kyle was weary of many of the new JavaScript features that are machine-centric; code features that will only be used by libraries and generators and never by an everyday programmer. Kyle insists that we should focus on developers first. Even WebAssembly and simimlar ideas are going to make web development a more complicated landscape to enter. Kyle started early and ended late. Further reading: Alan Kay, Douglas Englebart, Tom Dale.

"Cross-Platform Desktop Apps with Electron" (David Neal) was an introductory guide to using Electron, the cross-platform desktop UI technology behind Atom, VSCode, and the Slack desktop app. Starting in Electron seems easier than I expected. Architecture is similar to developing for the web, where we have server-side code and client-side code. It's better to make calls to the server than to run on the UI thread. Pretty much anything that you can install with npm can readily be used in Electron, including UI frameworks such as React and testing tools such as mocha.

I watched some lightning talks during lunch. Raspberry Pi celebrates their 6th year, something something Blockchain databases, jump-starting an open-source career via blogging or speaking, examples of unconscious bias in AI datasets, all the wrong ways to pronounce "kubectl", more on Red Hat's Istio service mesh framework, and ideas for replacing `docker` with other container tools.

"Framework Free – Building a Single Page Application Without a JS Framework" (Ryan Miller) described the way we used to make websites in 2013 without frameworks, but with all the nice HTML5 features. It's somewhat important to know how all of these things work under the covers, especially if you have to debug in the browser. It's not always necessary to have a big, hefty framework. I was somewhat horrified by the number of people in the audience who didn't know what jQuery was.

In "Intro to SVG", Tanner Hodges explained the basics of SVGs, when to use them, and when to seek alternatives. Interesting cases included textured content (which rendered significantly smaller as a PNG over an SVG), content that included text (which needed to be checked to verify that the text was rendered as native SVG elements), and photography (which, when rendered to SVG, literally included a data hash of the original image at high resolution, creating a massive file). He also touted the importance of new standards such as `webp` and `webv` when displaying content on the web.

In "WTH is JWT", Joel Lord broke down how JWTs are constructed by combining the encryption method, the payload (basic user information), and a secret key. Although the key can be deserialized and parsed to get to the information, the secret at the end is determined by hashing the key and the payload, so would-be attackers cannot simply change the JWT to gain access to the system without knowing the secret key. Of course, more security measures are necessary to keep intruders from gaining access, such as encrypted sessions and auth servers. The JWT Spec is still in the [proposal period](https://tools.ietf.org/html/rfc7519), so its definition may still change before finalization.

### Conclusion

Pretty good conference. I made a few interesting connections. I learned that a lot of people are in love with Vue right now, TypeScript is still popular, and microservices are the only way to build an application. My biggest complaint was that coffee was hard to come by.
