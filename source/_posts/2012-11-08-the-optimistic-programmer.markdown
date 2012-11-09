---
layout: post
title: "The Optimistic Programmer"
date: 2012-11-08 19:06
comments: true
categories: sep-blog-battle non-technical
---

Meet Anne. Anne's a programmer in a thriving tech company. Anne goes into work every day to work on a shiny new product for her company.

Anne's customer knows exactly what they want. Anne writes all of her tests using pure TDD. Anne updates her tasks on the company's ALM software, which was defined with programmers in mind and not just managers. She knows everything about her domain, and every UI element she touches is the epitome of good user experience. She works with 100 other programmers, each of which is equally as skilled as she is. Every time she pushes her most recent file changes, there are no merge conflicts and the build server works without a hitch. The deadline for the project is defined as whenever the last feature is finished. The best part: Anne spends fewer than an hour a week in meetings, none of which is a complete waste of time.

You've got to be kidding me.

If I experienced a day like I just described for Anne, I would have to assume I had been killed in a tragic car accident on my way into work.

How can a programmer possibly be optimistic? Let me say: The client never knows what they want. The programmer couldn't possibly write all of their tests TDD style unless they're a saint. ALM software has a tendency to suck. Programmers don't have the inherent ability to know precisely how a user will try to use the product. Oftentimes your teammates may not be on the same level as you are, or you may not be on as high a level as they are. There are always merge conflicts and build server anomalies. 'Random' is generally an apt description of project deadlines. Lastly, we spend lots of times in meetings which often don't appear to provide much face value. Those are the facts of life.

If we're optimists, why would we bother writing tests for anything beyond happy-path? How could we ever justify refactoring if we're optimistic that we did it right the first time? Why bother putting in security if no one will ever try to use their 1337 hax to get to the client's data? It's important that we stay planted firmly in reality to ensure we think of most everything that can go wrong when a customer uses a product.

I'm not saying there aren't optimistic moments in a programmer's life. Phrases like "I'm optimistic the project will end some day" and "I'm pretty sure the customer will like this" and "I highly doubt we'll get sued for that" are phrases that I'd feel inclined to throw around most any time to raise the morale of my companions.
