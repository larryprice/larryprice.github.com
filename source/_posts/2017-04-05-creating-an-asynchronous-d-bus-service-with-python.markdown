---
layout: post
title: "Creating an Asynchronous D-Bus Service with Python"
date: 2017-04-05 15:41:56 -0400
comments: true
categories: ubuntu python dbus
---

_This is the second in a series of blog posts on creating an asynchronous D-Bus service in python. For part 1, [go here](/blog/2017/04/04/creating-a-d-bus-service-with-python/)._

Last time we created a base for our asynchronous D-Bus service with a simple synchronous server/client. In this post, we'll start from that base which can be found on Github: [https://github.com/larryprice/python-dbus-blog-series/tree/part1](https://github.com/larryprice/python-dbus-blog-series/tree/part1). Of course, all of today's code can be found in the same project with the part2 tag: [https://github.com/larryprice/python-dbus-blog-series/tree/part2](https://github.com/larryprice/python-dbus-blog-series/tree/part2).

### Why Asynchronous? ###

Before we dive into making our service asynchronous, we need a reason to make our service asynchronous. Currently, our only d-bus object contains a single method, `quick`, which lives up to its namesake and is done very quickly. Let's add another method to `RandomData` which takes a while to finish its job.

``` python random_data.py
import dbus.service
import random
import time

class RandomData(dbus.service.Object):
    def __init__(self, bus_name):
        super().__init__(bus_name, "/com/larry_price/test/RandomData")
        random.seed()

    @dbus.service.method("com.larry_price.test.RandomData",
                         in_signature='i', out_signature='s')
    def quick(self, bits=8):
        return str(random.getrandbits(bits))

    @dbus.service.method("com.larry_price.test.RandomData",
                         in_signature='i', out_signature='s')
    def slow(self, bits=8):
        num = str(random.randint(0, 1))
        while bits > 1:
            num += str(random.randint(0, 1))
            bits -= 1
            time.sleep(1)

        return str(int(num, 2))
```

Note the addition of the `slow` method on the `RandomData` object. `slow` is a contrived implementation of building an n-bit random number by concatenating 1s and 0s, sleeping for 1 second between each iteration. This will still go fairly quickly for a small number of bits, but could take quite some time for numbers as low as 16 bits.

In order to call the new method, we need to modify our `client` binary. Let's add in the `argparse` module and take in a new argument: `--slow`. Of course, `--slow` will instruct the program to call `slow` instead of `quick`, which we'll add to the bottom of the program.

``` python client
#!/usr/bin/env python3

# Take in a single optional integral argument
import sys
import argparse

arg_parser = argparse.ArgumentParser(description='Get random numbers')
arg_parser.add_argument('bits', nargs='?', default=16)
arg_parser.add_argument('-s', '--slow', action='store_true',
                        default=False, required=False,
                        help='Use the slow method')

args = arg_parser.parse_args()

# Create a reference to the RandomData object on the  session bus
import dbus, dbus.exceptions
try:
    bus = dbus.SessionBus()
    random_data = bus.get_object("com.larry-price.test", "/com/larry_price/test/RandomData")
except dbus.exceptions.DBusException as e:
    print("Failed to initialize D-Bus object: '%s'" % str(e))
    sys.exit(2)

# Call the appropriate method with the given number of bits
if args.slow:
    print("Your random number is: %s" % random_data.slow(int(args.bits)))
else:
    print("Your random number is: %s" % random_data.quick(int(args.bits)))

```

Now we can run our `client` a few times to see the result of running in slow mode. Make sure to start or restart the `service` binary before running these commands:

``` bash
$ ./client 4
Your random number is: 2
$ ./client 4 --slow
Your random number is: 15
$ ./client 16
Your random number is: 64992
$ ./client 16 --slow
Traceback (most recent call last):
  File "./client", line 26, in <module>
    print("Your random number is: %s" % random_data.slow(int(args.bits)))
  File "/usr/lib/python3/dist-packages/dbus/proxies.py", line 70, in __call__
    return self._proxy_method(*args, **keywords)
  File "/usr/lib/python3/dist-packages/dbus/proxies.py", line 145, in __call__
    **keywords)
  File "/usr/lib/python3/dist-packages/dbus/connection.py", line 651, in call_blocking
    message, timeout)
dbus.exceptions.DBusException: org.freedesktop.DBus.Error.NoReply: Did not receive a reply. Possible causes include: the remote application did not send a reply, the message bus security policy blocked the reply, the reply timeout expired, or the network connection was broken.
```

Your mileage may vary (it _is_ a random number generator, after all), but you should eventually see a similar crash which is caused by a timeout in the response of the D-Bus server. We know that this algorithm works; it just needs more time to run. Since a synchronous call won't work here, we'll have to switch over to more asynchronous methods...

### An Asynchronous Service ###

At this point, we can go one of two ways. We can use the `threading` module to spin threads within our process, or we can use the `multiprocessing` module to create child processes. Child processes will be slightly pudgier, but will give us more functionality. Threads are a little simpler, so we'll start there. We'll create a class called `SlowThread`, which will do the work we used to do within the `slow` method. This class will spin up a thread that performs our work. When the work is finished, it will set a `threading.Event` that can be used to check that the work is completed. `threading.Event` is a cross-thread synchronization object; when the thread calls `set` on the `Event`, we know that the thread is ready for us to check the result. In our case, we call `is_set` on our event to tell a user whether or not our data is ready.

``` python random_data.py
# ...

import threading
class SlowThread(object):
    def __init__(self, bits):
        self.finished = threading.Event()
        self.result = ''

        self.thread = threading.Thread(target=self.work, args=(bits,))
        self.thread.start()
        self.thread_id = str(self.thread.ident)

    @property
    def done(self):
        return self.finished.wait(1)

    def work(self, bits):
        num = str(random.randint(0, 1))
        while bits > 1:
            num += str(random.randint(0, 1))
            bits -= 1
            time.sleep(1)

        self.result = str(num)
        self.finished.set()

# ...
```

On the `RandomData` object itself, we'll initialize a new thread tracking list called `threads`. In `slow`, we'll initialize a `SlowThread` object, append it to our `threads` list, and return the thread identifier from `SlowThread`. We'll also want to add a method to try to get the result from a given `SlowThread` called `slow_result`, which will take in the thread identifier we returned earlier and try to find the appropriate thread. If the thread is finished (the `event` is set), we'll remove the thread from our list and return the result to the caller.

``` python random_data.py
# ...

class RandomData(dbus.service.Object):
    def __init__(self, bus_name):
        super().__init__(bus_name, "/com/larry_price/test/RandomData")

        random.seed()
        self.threads = []

    # ...

    @dbus.service.method("com.larry_price.test.RandomData",
                         in_signature='i', out_signature='s')
    def slow(self, bits=8):
        thread = SlowThread(bits)
        self.threads.append(thread)
        return thread.thread_id

    @dbus.service.method("com.larry_price.test.RandomData",
                         in_signature='s', out_signature='s')
    def slow_result(self, thread_id):
        thread = [t for t in self.threads if t.thread_id == thread_id]
        if not thread:
            return 'No thread matching id %s' % thread_id

        thread = thread[-1]
        if thread.done:
            result = thread.result
            self.threads.remove(thread)
            return result

        return ''
```

Last thing we need to do is to update the client to use the new methods. We'll call `slow` as we did before, but this time we'll store the intermediate result as the thread identifier. Next we'll use a while loop to spin forever until the result is ready.

``` python client
# ...

if args.slow:
    import time
    thread_id = random_data.slow(int(args.bits))
    while True:
        result = random_data.slow_result(thread_id)
        if result:
            print("Your random number is: %s" % result)
            break
        time.sleep(1)

# ...
```

Note that this is not the smartest way to do this; more on that in the next post. Let's give it a try!

``` bash
$ ./client 4
Your random number is: 7
$ ./client 4 --slow
Your random number is: 12
$ ./client 16
Your random number is: 5192
$ ./client 16 --slow
27302
```

### Next time ###

This polling method works as a naive approach, but we can do better. Next time we'll look into using D-Bus signals to make our client more asynchronous and remove our current polling implementation.

As a reminder, the end result of our code in this post is MIT Licensed and can be found on Github: [https://github.com/larryprice/python-dbus-blog-series/tree/part2](https://github.com/larryprice/python-dbus-blog-series/tree/part2).
