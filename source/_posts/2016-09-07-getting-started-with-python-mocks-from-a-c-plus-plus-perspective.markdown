---
layout: post
title: "Getting Started with Python Mocking and Patching"
date: 2016-09-07 23:00:09 -0400
comments: true
categories: python testing
---

I currently write a lot of python and C++. Although I religiously unit test my C++ code, I'm a bit ashamed to say that I haven't had much experience with python unit testing until recently. You know how it is - python is one of those interpreted languages, you mostly use it to do quick hacks, it doesn't _need_ tests. Until you've written your entire D-Bus service using python, and every time you make a code change _a literal python_ appears on the screen to crash your computer. So I've started writing a bunch of tests and found (as expected) a tangled mess of dependencies and system calls.

In many C-like languages, you can fix most of your dependency problems with [The Big Three](https://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing#answer-346440): mocks, fakes, and stubs. A fake is an actual implementation of an interface used for non-production environments, a stub is an implementation of an interface returning a pre-conceived result, and a mock is a wrapper around an interface allowing a programmer to accurately map what actions were performed on the object. In C-like languages, you use [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) to give our classes fakes, mocks, or stubs instead of real objects during testing.

The good news is that we can also use dependency injection in python! However, I found that relying solely on dependency injection would pile on more dependencies than I wanted and was not going to work to cover all my system calls. But python is a dynamic language. In python, you can literally change the definition of a class inside of another class. We call this operation __patch__ and you can use it extensively in testing to do some pretty cool stuff.

### Code Under Test ###

Let's define some code to test. For all of these examples, I'll be using python3.5.2 with the [unittest](https://docs.python.org/3/library/unittest.html) and [unittest.mock](https://docs.python.org/3/library/unittest.mock.html) libs on Ubuntu 16.10. You can the final versions of these code samples [on github](https://github.com/larryprice/python-mocks-blog-post).

``` python
from random import randint

class WorkerStrikeException(Exception):
    pass

class Worker(object):
    """
    A Worker will work a full 40 hour week and then go on strike. Each time
    a Worker works, they work a random amount of time between 1 and 40.
    """
    def __init__(self):
        self.hours_worked = 0

    def work(self):
        timesheet = randint(1, 40)
        self.hours_worked += timesheet
        if self.hours_worked > 40:
            raise WorkerStrikeException("This worker is picketing")

        return timesheet

class Boss(object):
    """
    A Boss makes profit using workers. Bosses squeeze 1000 monies out of a
    Worker for each hour worked. Workers on strike are instantly replaced.
    """
    def __init__(self, worker):
        self.worker = worker
        self.profit = 0

    def make_profit(self):
        try:
            self.profit += self.worker.work()*1000
        except WorkerStrikeException as e:
            print("%s" % e)
            self.worker = Worker()
            self.profit += self.worker.work()*1000
        finally:
            return self.profit
```

These are two simple classes (and a custom `Exception`) that we'll use to demonstrate unit testing in python. The first class, `Worker`, will work a maximum of 40 hours per week before picketing it's corporation. Each time `work` is called, the `Worker` will work a random number of hours. The `Boss` class takes in a `Worker` object, which it uses as it performs `make_profit`. The profit is determined by the number of hours worked multiplied by 1000. When the worker starts picketing, the `Boss` will hire a new `Worker` to take their place. So it goes.

### Mocking the Worker Class ###

Our goal is to fully test the `Boss` class. We've left ourselves a dependency to inject in the `__init__` method, so we could start there. We'll mock the `Worker` and pass it into the `Boss` initializer. We'll then set up the `Worker.work` method to always return a known number so we can test the functionality of `make_profit`.

``` python
import unittest.mock
from unittest import TestCase

from corp import work  # your impl file

class BossTest(TestCase):
    def test_profit_adds_up(self):
        worker = unittest.mock.create_autospec(work.Worker)
        worker.work.return_value = 8
        boss = work.Boss(worker)
        self.assertEqual(boss.make_profit(), 8000)
        self.assertEqual(boss.make_profit(), 16000)
        worker.work.return_value = 10
        self.assertEqual(boss.make_profit(), 26000)

        worker.work.assert_has_calls([
            unittest.mock.call(),
            unittest.mock.call(),
            unittest.mock.call()
        ])

if __name__ == '__main__':
    unittest.main()
```

To run this test, use the command `python3 -m testtools.run test`, where `test` is the name of your test file without the `.py`.

One curiosity here is `unittest.mock.create_autospec`. Python will also let you directly create a `Mock`, which will absorb all attribute calls regardless of whether they are defined, and `MagicMock`, which is like `Mock` except it also mocks magic methods. `create_autospec` will create a mock with all of the defined attributes of the given class (in our case `work.Worker`), and raise an Exception when the attribute is not defined on the specced class. This is really handy, and eliminates the possibility of tests "accidentally passing" because they are calling default attributes defined by the generic `Mock` or `MagicMock` initializers.

We set the return value of the `work` function with `return_value`, and we can change it on a whim if we so desire. We then use `assertEqual` to verify the numbers are crunching as expected. One further thing I've shown here is `assert_has_calls`, a mock assertion to verify that `work` was called 3 times on our mock method.

You may also note that we subclassed `TestCase` to enable running this class as part of our unit testing framework with the special `__main__` method definition at the bottom of the file.

### Patching the Worker Class ###

Although our first test demonstrates how to `make_profit` with a happy worker, we also need to verify how the `Boss` handles workers on strike. Unforunately, the `Boss` class creates his own `Worker` internally after learning they can't trust the `Worker` we gave them in the initializer. We want to create consistent tests, so we can't rely on the random numbers generated by `randint` in `Worker.work`. This means we can't just depend on dependency injection to make these tests pass!

At this point we have two options: we can patch the `Worker` class or we can patch the `randint` function. Why not both! As luck would have it, there are a few ways to use [`patch`](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch), and we can explore a couple of these ways in our two example tests.

We'll patch the `randint` function using a method decorator. Our intent is to make `randint` return a static number every time, and then verify that profits keep booming even as we push workers past their limit.

``` python
@unittest.mock.patch('corp.work.randint', return_value=20)
def test_profit_adds_up_despite_turnover(self, randint):
    boss = work.Boss(work.Worker())
    self.assertEqual(boss.make_profit(), 20000)
    self.assertEqual(boss.make_profit(), 40000)
    self.assertEqual(boss.make_profit(), 60000)
    self.assertEqual(boss.make_profit(), 80000)

    randint.assert_has_calls([
        unittest.mock.call(1, 40), unittest.mock.call(1, 40),
        unittest.mock.call(1, 40), unittest.mock.call(1, 40)
    ])
```

When calling `patch`, you must describe the namespace relative to the module you're importing. In our case, we're using `randint` in the `corp.work` module, so we use `corp.work.randint`. We define the `return_value` of `randint` to simply be 20. A fine number of hours per day to work an employee, according to the `Boss`. `patch` will inject a parameter into the test representing an automatically created mock that will be used in the patch, and we use that to assert that our calls were all made the way we expected.

Since we know the inner workings of the `Worker` class, we know that this test exercised our code by surpassing a 40-hour work week for our poor `Worker` and causing the `WorkerStrikeException` to be raised. In doing so, we're depending on the `Worker`/`Boss` implementation to stay in-sync, which is a dangerous assumption. Let's explore patching the `Worker` class instead.

To spice things up, we'll use the `ContextManager` syntax when we patch the `Worker` class. We'll create one mock `Worker` outside of the context to use for dependency injection, and we'll use this mock to `raise` the `WorkerStrikeException` as a side effect of `work` being called too many times. Then we'll patch the `Worker` class for newly created instances to return a known timesheet.

``` python
def test_profit_adds_up_despite_strikes(self):
    worker = unittest.mock.create_autospec(work.Worker)
    worker.work.return_value = 12
    boss = work.Boss(worker)

    with unittest.mock.patch('corp.work.Worker') as MockWorker:
        scrub = MockWorker.return_value
        scrub.work.return_value = 4

        self.assertEqual(boss.make_profit(), 12000)
        self.assertEqual(boss.make_profit(), 24000)

        worker.work.side_effect = work.WorkerStrikeException('Faking a strike!')
        self.assertEqual(boss.make_profit(), 28000)
        self.assertEqual(boss.make_profit(), 32000)

        worker.work.assert_has_calls([
            unittest.mock.call(), unittest.mock.call(), unittest.mock.call()
        ])
        scrub.work.assert_has_calls([
            unittest.mock.call(), unittest.mock.call()
        ])
```

After the first `Worker` throws a `WorkerStrikeException`, the second `Worker` (scrub) comes in to replace them. In patching the `Worker`, we are able to more accurately describe the behavior of `Boss` regardless of the implementation details behind `Worker`.

### A Non-Political Conclusion ###

I'm not saying this is the best way to go about unit testing in python, but it is an option that should help you get started unit testing legacy code. There are certainly those who see this level of micromanaging mocks and objects as tedious, but there is be benefit to defining the way a class acts under exact circumstances. This was a contrived example, and your code may be a little bit harder to wrap with tests.

Now you can go get Hooked on Pythonics!
