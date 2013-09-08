---
layout: post
title: "Code Complete Second Edition"
date: 2013-02-19 22:38
comments: true
categories: write-ups non-technical
---

> Design is a process of carefully planning small mistakes in order to avoid making big ones.
> -- Steve McConnell

###The Gist

_[Code Complete Second Edition][cc]_ by Steve McConnell is the ultimate programmer's handbook, though certainly not a pocket guide considering its massive 850+ page size. This book contains a seemingly endless amount of information regarding the state of programming circa 2004. Topics range from extremely technical, such as making code readable above making it clever, to office politics, such as dealing with non-technical managers.

[cc]: http://www.amazon.com/gp/product/0735619670/ref=as_li_tf_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0735619670&linkCode=as2&tag=larpriandthee-20

###My Opinion

McConnell has written an encyclopedia for software development. In doing so, the book sometimes suffers from the "wide as an ocean, shallow as a puddle" complex. Nonetheless, the book holds an incredible amount of information that I found refreshing to see in print.

The use of the term "construction" to refer to coding throughout the book is an apt analogy for McConnell's views on the world of software development. Before beginning construction, a team should have a plan. Construction requires a solid base. While constructing a project, individuals should ensure that anyone else could look at their part of the project and figure out what's happening. Working with a partner often improves quality. People doing construction should test their work to ensure the integrity of their product. Redoing lower levels causes a lot of pain. The team is unlikely to hit the original due date. Etc, etc.

Many things in this book go directly against what I was taught in at university. McConnell recites a quote several times in the book:

> Debugging is twice as hard as writing the code in the first place. Therefore, if you write the code as cleverly as possible, you are, by definition, not smart enough to debug it.
> -- Brian Kernighan

At university, we're taught all the secrets of writing clever code: pointer manipulation, recursion, inappropriate variable reuse, and others. We were also never taught to plan ahead (...besides flowcharts, which were never required) or test, both of which McConnell highly recommends as necessary steps in software development. It's no wonder my programs took so long to debug! Of course, I've learned to perform these tasks naturally while working as a Real Life Programmer. Reading this book is likely more helpful than taking the first few semesters of programming at university.

A small issue I had was a wishy-washy attitude towards comments. In one chapter, McConnell describes having minimal comments in a code and keeping it mostly self-documenting. He then goes on to show all kinds of different, horrifying comments and justifies why they are sometimes okay.

This is where I will note that _Code Complete Second Edition_ is a book published by [Microsoft Press][mp]. I was often surprised by McConnell's adamance that Visual Basic is the most popular programming language at the time of publishing. Even circa 2004 (the year this book was published), the [Tiobe index][tiobe] shows VB behind Java, C, and C++. The [PyPl index][pypl] also shows VB behind Java, PHP, C, and C++ in 2004. For the record, VB is a silly language.

[tiobe]: http://www.tiobe.com/index.php/content/paperinfo/tpci/index.html
[pypl]: https://sites.google.com/site/pydatalog/pypl/PyPL-PopularitY-of-Programming-Language
[mp]: http://en.wikipedia.org/wiki/Microsoft_Press

###Who Would Like This

This book would be nice required reading for students. Given the length and technical depth, it's practically a textbook. Developers a few years out of university would still enjoy this text, but some topics are so obvious or overdone that many readers may start skipping chapters. If a developer knew of precisely the area he or she wanted to improve, then said developer could likely benefit from perusing a chapter of this book covering that topic.
