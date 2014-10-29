---
layout: post
title: "My Battle With Hour-Based Estimation"
date: 2014-10-29 18:33:27 -0400
comments: true
categories: non-technical agile
---

I despise estimating in hours. Hours are too granular. Hours are too difficult to distinguish from hours on a clock (_they're even spelled the same_).

In any project where we work in hours, we always desparately task out stories in hours fill our capacity. _Which is also measured in hours_. We take people's days off and we estimate that developers spend about 3/4 of the day watching Youtube videos and reading Hacker News. So a developer works __about 6 clock hours__ in a given day.

Then we start tasking out stories. We estimate that this task will take __about 12 hours__, this one __about 8 hours__, and we'll top it all off with a task of __about 4 hours__ for ad-hoc testing. All in all, this story is estimated to take __exactly 24 hours__.

Now we do some math. Obviously, since a developer can get __6 hours__ of work done in a day and this story will take __24 hours__, the story will be done in __exactly 4 development days__, or __exactly 3 real days__. We watch our capacity (in hours) fill by 24 and note all the space that's left.

Someone without reference to our developer-day-to-work-day comparison walks by. Excellent, it'll be done in three days, you say? That fits the schedule perfectly, tell the client we'll be done with that feature by Thursday. Let's add some more to your backlog...

For all we know, this estimate may work out. However, from my experience, the 8 hour task and 12 hour task will probably spend about the same amount of time in development, taking anywhere from 1-3 days each. Fortunately, the "testing" task will only take about 30 minutes.

What's wrong here?

Humans are bad at estimating. We become much worse at estimating when we start talking about knowledge work. I happen to believe we become even worse at estimating when we try to use time as a unit of measure. How long does it take you to get to work? I would say it takes me about 5 minutes. Google Maps tells me I would be lucky to get there in 10 without traffic. My estimation in minutes is __bad__, conflated by the monotony of performing a menial task over and over again.

When we have retros, we don't discuss how incorrect our hourly measurements were. We have a hybrid system of estimating stories in points and tasks in hours. We discuss the discrepancy between estimated and actual story points instead, despite the disconnect between our task estimates and our story estimates. (Another reason we don't discuss hour estimations at the retro is that it would take __forever__ due to the high number of tasks and the granularity of hours (did this take one day or two, I can't remember)).

Do I have a solution to this problem? Of course I do. Don't use hours. Stick with points, and keep them as vague and hand-wavey as possible.

We often run into an issue of associating points with number of hours or days. "Well, a 3 takes us about a week and a 5 takes us about 2 weeks. This story seems like it'll take about a week."

Points __do not__ translate directly to how long a story will take. Points __do__ translate to how much work a story feels like relative to the stories which have come before it.

Eventually, you should have a big enough pile of 1s, 2s, 3s, and 5s that you can relate a new story to a previous story. This story feels like a 3, about as much work as that story where we had to herd those cats. This story feels like a 1, technically all I have to do is flip this boolean...

When starting a project, estimate stories but don't try to estimate a capacity. Just see how many points you can get done in a sprint. Do it again for the second sprint. Keep doing it until your numbers balance out. This means that your estimates are becoming consistent and you're getting over the new project hump. Now you can tell your manager that you should be able to get 20 points done in a sprint, because you've consistently bounced around that number the past few sprints.

Don't get too hasty and start telling people that you should be able to do more points with each sprint. I hate to break it to you, but you're not getting that much better each sprint. You need to adjust your point estimation to match your current competency level.

End rant. It'll probably be a long time before we see the death of hourly estimations, especially with big-name tools like Rally and Visual Studio Online containing fields explicitly for that purpose. Just fight the good fight for now.
