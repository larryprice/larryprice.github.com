---
layout: post
title: "Effective Modern C++"
date: 2016-06-29 20:35:23 -0400
comments: true
categories: c++ write-ups
---

### The Gist ###

How can one man give the world so much? Scott Meyers transformed my understanding of C++ with [_Effective C++_](/blog/2013/07/16/effective-c-plus-plus/), a book which not only teaches good C++ practices and principles, but also explains what's going on behind the scenes to make those efforts so effectual. [_Effective Modern C++_](http://amzn.to/29dU9FO) is the same book aimed at a different audience. The audience of _Effective C++_ was the developer who could use C++ to build a humble home of straw or wood, but didn't know that C++ was born to create homes of brick and mortar. _Effective Modern C++_ is for the seasoned developer who knows the brick home they built with C++98 can stand tall, but is bewildered by the plethora of modern amenities available in the top-floor penthouse that is C++11.

### Takeaways ###

Even when first introduced to the language, it seemed the mentality around C++ was that it was simple, baremetal, and robust. It didn't need garbage collection. It didn't need decent threading (fork that). Lambdas? Type deduction? Go talk to a language specification who cares!

But here we are. C++11 can now act a little bit more like its cousins C# and Java, but run fast like its pappy C. I have been on a number of projects where C++11 is king for the last 3 months. The last time I had been on a C++ project, we seemed to be stuck in the ice age. As I first started reading this book, I would read one of the items and apply it directly to the code I was working on literally the following morning. This new C++ is downright _luxurious_ compared to the old one.

C++11 gives us the `auto` pointer for type deduction, similar to `var` in C#. Some examples:

``` c++
auto x = 1;                 // x is int
auto y = new Thing();       // y is Thing*
const auto z = "the thing"; // z is a const char*
```

C++11 also has new loop syntax for iterators:

``` c++
std::vector<int> v{4, 6, 0, 3, 3};

for (const auto& value: v)
{
	std::cout << value;
}
std::cout << std::endl
// prints 46033
```

You may notice in the above code the use of curly braces to initalize the std::vector. Braced initialization allows us to use a `std::initalizer_list` to initalize STL objects, but it also allows basic construction.

There are now lambdas: function handles defined dynamically which can capture other variables (called a closure).

``` c++
auto x = 5;
auto my_func = [&x](int y) {return x+y;};

my_func(5); // 10
my_func(3); // 8
x = 11;
my_func(5); // 16
```

In the above example, x is captured by reference. You could also copy-capture x by excluding the `&`.

You like garbage collection? We got you covered. There's `std::unique_ptr` to represent one-shot memory that should be deleted when the pointer goes out of scope, and there's `std::shared_ptr` which is reference counted and will be deleted when all references to the shared_ptr go out of scope. These language enhancements are essential, and anyone well-versed in the usage of `boost::scoped_ptr` and `boost::shared_ptr` will have no trouble getting the hang of these.

The concurrency API is pretty neat, though I haven't had much chance to play with it. `std::atomic` allows you to create objects which are guaranteed to be read/write thread-safely. `std::future` allows you to fire off a command in another thread and continue after it's completion.

Looking to override a method? You can indicate an intentional override with the `override` keyword to tell the reader and the compiler you're intending to override a parent method. Comes in handy.

Another nice construct is `nullptr`. In C++, `NULL` is actually just 0. Because of this you might even see your fellow developers comparing pointers to 0 while you try to determine their intent. We can now compare and set our NULL pointers to `nullptr`: an improvement soft in functionality, but noticeable in readability.

Although I loved reading about C++ in bite-sized chunks throughout this book, there were a few things that went over my head (not just the first time). A major point of friction between myself and the author were universal references (`&&`) and the `move` operator. These concepts were new to me and difficult to grasp the first few times they were brought up in this book. It may have been the order they were presented, or it may have been my lack of contact with them in the real world, but I would recommend having some level of understanding for universal references before reading those parts of this book.

The `move` operation moves the contents of memory from one object into another, as opposed to a copy operation which will duplicate that memory. A universal reference is, in some sense, a way to allow either a `copy` or `move` to be called based on whether an lvalue or an rvalue is being passed in. There are lots of rules involved, and sometimes `move` is faster than `copy` but other times it isn't. For me (and I assume many others), this confusion will lead me to largely ignore this feature for now.

There is also quite a bit of discussion early in the book about using type deduction in templates with `auto` and `decltype`, but this discussion made my head hurt and made me glad I don't do much template metaprogramming.

On top of all this goodness (and more I didn't mention), C++14 includes a lot of bonus features that make C++ even a little sweeter. Here's a [shortlist](https://en.wikipedia.org/wiki/C%2B%2B14). (Looking for a list of C++11 features? [Here you go](https://en.wikipedia.org/wiki/C%2B%2B11).)

### Action Items ###

* Read [Thomas Becker's overview of universal references](http://thbecker.net/articles/rvalue_references/section_01.html)
* Buy the _Effective_ books missing from my collection
* Keep coding with C++
* Enable C++14 in a project
