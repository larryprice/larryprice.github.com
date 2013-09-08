---
layout: post
title: "Effective C++"
date: 2013-07-16 22:11
comments: true
categories: write-ups
---

###The Gist

[_Effective C++_](http://www.amazon.com/gp/product/0321334876/ref=as_li_ss_tl?ie=UTF8&camp=1789&creative=390957&creativeASIN=0321334876&linkCode=as2&tag=larpriandthee-20) is a list of ways to write not-your-average-bear C++ code. The author, [Scott Meyers](http://www.aristeia.com/), outlines 8 specific topics for improving code, and a few more miscellaneous tips.

###My Opinion

Lots of good can come from reading this book. I borrowed it from a more-experienced co-worker and noticed many of the interesting coding standards this co-worker used came directly from this book. And I agree with it. I've been actively trying to follow the lessons learned in this book from the moment I picked it up.

I learned C++ the hard way: I was thrown into an internship where I didn't really have enough skills to contribute to the project, and my "boss" didn't really have anywhere to put me. So I was handed a book, pointed to [this wonderous website](http://www.cplusplus.com/), and I wrote some little helper apps for the project. I wasn't taught anything about form or any of that formal business. Then I took a C++ class in college, and never bothered to learn any of the coding standards because I had enough previous knowledge to get by my own way (and appear to be really good while doing it).

So this book showed me a lot of the why certain things in C++ are the way they are. It explained to me that a copy constructor is called every time you pass an object directly through a function, and how assignment operators are used whenever an equals sign appears. It told me why my destructors need to be virtual. It explained a lot about smart_ptr and auto_ptr that I never thought to think about.

Admittedly, there are parts of the book that are hard to follow. I consider Chapter 7 (Templates and Generic Programming) and Chapter 8 (Customizing new and delete) to be extremely technical chapters not meant for the faint of heart, and the lessons learned in these chapters are not necessarily useful in day-to-day programming. Chapter 9 is simply called Miscellany, and it delivers no new information for improving one's C++ skills.

###Who Would Like This

C++ Programmers should have this book (at least the first 6 chapters) crammed down their throats and into their squishy pink brains. After learning all the "basic" concepts of C++, the lessons this book teaches in design and the nature of C++ should be taken into consideration.
