---
layout: post
title: "Design for Real Life (Writeup)"
date: 2016-03-24 22:00:47 -0400
comments: true
categories: write-ups
---

### The Gist

Have you ever tried to buy airline tickets in a hurry? Have you ever had to find directions to the ER on the hospital website during an actual emergency? Have you ever wondered why you have to give a bug tracker your "title," "gender," or "address"? [Design for Real Life](https://abookapart.com/products/design-for-real-life) written by [Eric Meyer](http://meyerweb.com/) and [Sara Wachter-Boettcher](https://www.sarawb.com/) explores the pitfalls of designing for the perfect user and the dangers of asking unnecessary or inappropriate questions.

By the way, the means by which I found out this great book existed is by following Sara on Twitter: [you should too](https://twitter.com/sara_ann_marie)!

### Takeaways

We spend a lot of time designing our websites for the perfect persona. Right this minute, you probably have Sam the Salesperson, Hermoine the HR Rep, and Monica the Manager all smiling down upon you from their painter's tape thrones on the wall next to your cubicle. You meet these caricatures at their best: bellies full of coffee, heads full of false worries, plenty of time, and a healthy dose of get-up-and-go in their web browsing digits. These idealized characters are a great way to make a best-case-scenario product, really nailing that "90%"of users everyone wants to find.

What about the other 10% of users? At the start of a long and miserable day full of meetings, can someone short on time and patience create yet another calendar event in the few short minutes they have available? Thirty seconds before Polly the Procrastinator's timesheet is due, can she quickly add in her time without filling in meaningless fields? Instead of focusing on the ideal case, we might be better-served using this kind of time-crunched, ambivalent user as one of our personae. By designing for the streamlined user, we'll likely find that we've also satisfied the needs of our "ideal" users.

We ask for too much information. Think about how many times you've just filled in junk information in forms to get through them as fast as possible. It's not shameful; it just wasn't worth your time to fill our information that you knew the application wouldn't use or could potentially use against you. On another note, think about how many times you've given up filling out a form for a website because it was just too long. Asking for too much information causes user fatigue, and you are guaranteed to lose users when you start asking them for too much. If your website doesn't need to take location, title, or city of birth to get its job done, stop asking for them. If you have a tough time removing these fields, then write some user-visible text explaining why you need this information.

Some questions could cause an unintended emotional response from users. If you're going through a tough time in your marriage or a recent divorce, filling out "title" or "marital status" could cause unwanted emotional duress. A "gender" dropdown with two options is non-inclusive to users with a non-binary gender identity, and may cause users going through a transition to wince at such a question. Someone who has just suffered the loss of a parent, child, or sibling could feel great emotional pain when you, a complete stranger, bring up family members for no reason but to feed to your bottomless data pit. Once again, the first question you ask yourself should be whether or not you absolutely need that information. Again, if you really want it, consider making more fields optional and explaining how this information will help your application.

Do more user interviews before implementing newly-designed features. We can't assume that everyone will use the app the way we think they will, and we'll always be better off for getting more information.

Your app probably doesn't need to be funny. We litter our error messages with "Whoopsie!" and our log out messages with "Sayonara, sucker!", but this is an absolute waste. When errors occur, there's a chance you've just angered your user. You know what makes them angrier? That error message you wrote that you found funny at the time. Be informative and supportive in your messaging, and your users will be better off for it.

### Action Items

* Stop trying to be funny in the copy. It's only funny when developing. Maybe we should have some sort of i18n setting that shows funny messages during development and English in production.
* Don't ask so many questions.
* More form fields should be open-answerable. Also more form fields should be optional. Also see previous bullet.
* Streamline the hard stuff. It may not be the most-used feature, but if it's important to a user in emergency mode you can bet it'll be appreciated.
