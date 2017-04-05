---
layout: post
title: "Creating a D-Bus Service with Python"
date: 2017-04-04 21:40:49 -0400
comments: true
categories: ubuntu python dbus async
---

I've been working on a d-bus service to replace some of the management guts of my project for a while now. We started out creating a simple service, but some of our management processes take a long time to run, causing a timeout error when calling these methods. I needed a way to run these tasks in the background and report status to any possible clients. I'd like to outline my approach to making this possible. This will be a multi-part blog series starting from the bottom: a very simple, synchronous d-bus service. By the end of this series, we'll have a small codebase with asynchronous tasks which can be interacted with (input/output) from D-Bus clients.

All of this code is written with python3.5 on Ubuntu 17.04 (beta), is MIT licensed, and can be found on Github: [https://github.com/larryprice/python-dbus-blog-series/tree/part1](https://github.com/larryprice/python-dbus-blog-series/tree/part1).

### What is D-Bus? ###

From Wikipedia:

> In computing, D-Bus or DBus (for "Desktop Bus"), a software bus, is an inter-process communication (IPC) and remote procedure call (RPC) mechanism that allows communication between multiple computer programs (that is, processes) concurrently running on the same machine.

D-Bus allows different processes to communicate indirectly through a known interface. The bus can be system-wide or user-specific (session-based). A D-Bus service will post a list of available objects with available methods which D-Bus clients can consume. It's at the heart of much Linux desktop software, allowing processes to communicate with one another without forcing direct dependencies.

### A synchronous service ###

Let's start by building a base of a simple, synchronous service. We're going to initialize a loop as a context to run our service within, claim a unique name for our service on the session bus, and then start the loop.

``` python service
#!/usr/bin/env python3

import dbus, dbus.service, dbus.exceptions
import sys

from dbus.mainloop.glib import DBusGMainLoop
from gi.repository import GLib

# Initialize a main loop
DBusGMainLoop(set_as_default=True)
loop = GLib.MainLoop()

# Declare a name where our service can be reached
try:
    bus_name = dbus.service.BusName("com.larry-price.test",
                                    bus=dbus.SessionBus(),
                                    do_not_queue=True)
except dbus.exceptions.NameExistsException:
    print("service is already running")
    sys.exit(1)

# Run the loop
try:
    loop.run()
except KeyboardInterrupt:
    print("keyboard interrupt received")
except Exception as e:
    print("Unexpected exception occurred: '{}'".format(str(e)))
finally:
    loop.quit()
```

Make this binary executable (`chmod +x service`) and run it. Your service should run indefinitely and do... nothing. Although we've already written a lot of code, we haven't added any objects or methods which can be accessed on our service. Let's fix that.

``` python dbustest/random_data.py
import dbus.service
import random

class RandomData(dbus.service.Object):
    def __init__(self, bus_name):
        super().__init__(bus_name, "/com/larry_price/test/RandomData")
        random.seed()

    @dbus.service.method("com.larry_price.test.RandomData",
                         in_signature='i', out_signature='s')
    def quick(self, bits=8):
        return str(random.getrandbits(bits))
```

We've defined a D-Bus object `RandomData` which can be accessed using the path `/com/larry_price/test/RandomData`. This style of string is the general style of an object path. We've defined an interface implemented by `RandomData` called `com.larry_price.test.RandomData` with a single method `quick` as declared with the `@dbus.service.method` context decorator. `quick` will take in a single parameter, `bits`, which must be an integer as designated by the `in_signature` in our context decorator. `quick` will return a string as specified by the `out_signature` parameter. All that `quick` does is return a random string given a number of bits. It's simple and it's fast.

Now that we have an object, we need to declare an instance of that object in our service to attach it properly. Let's assume that `random_data.py` is in a directory `dbustest` with an empty `__init__.py`, and our service binary is still sitting in the root directory. Just before we start the loop in the `service` binary, we can add the following code:

``` python service
# ...
# Run the loop
try:
    # Create our initial objects
    from dbustest.random_data import RandomData
    RandomData(bus_name)

    loop.run()
# ...
```

We don't need to do anything with the object we've initialized; creating it is enough to attach it to our D-Bus service and prevent it from being garbage collected until the service exits. We pass in `bus_name` so that `RandomData` will connect to the right bus name.

### A synchronous client ###

Now that you have an object with an available method on our service, you're probably interested in calling that method. You can do this on the command line with something like `dbus-send`, or you could find the service using a GUI tool such as `d-feet` and call the method directly. But eventually we'll want to do this with a custom program, so let's build a very small program to get started.

``` python client
#!/usr/bin/env python3

# Take in a single optional integral argument
import sys
bits = 16
if len(sys.argv) == 2:
    try:
        bits = int(sys.argv[1])
    except ValueError:
        print("input argument must be integer")
        sys.exit(1)

# Create a reference to the RandomData object on the  session bus
import dbus, dbus.exceptions
try:
    bus = dbus.SessionBus()
    random_data = bus.get_object("com.larry-price.test", "/com/larry_price/test/RandomData")
except dbus.exceptions.DBusException as e:
    print("Failed to initialize D-Bus object: '%s'" % str(e))
    sys.exit(2)

# Call the quick method with the given number of bits
print("Your random number is: %s" % random_data.quick(bits))
```

A large chunk of this code is parsing an input argument as an integer. By default, `client` will request a 16-bit random number unless it gets a number as input from the command line. Next we spin up a reference to the session bus and attempt to find our `RandomData` object on the bus using our known service name and object path. Once that's initialized, we can directly call the `quick` method over the bus with the specified number of bits and print the result.

Make this binary executable also. If you try to run `client` without running `service`, you should see an error message explaining that the `com.larry-price.test` D-Bus service is not running (which would be true). Start `service`, and then run `client` with a few different input options and observe the results:

``` bash
$ ./service & # to kill service later, be sure to note the pid here!
$ ./client
Your random number is: 41744
$ ./client 100
Your random number is: 401996322348922753881103222071
$ ./client 4
Your random number is: 14
$ ./client "new donk city"
input argument must be integer
```

That's all there is to it. A simple, synchronous server and client. The server and client do not directly depend on each other but are able to communicate unidirectionally through simple method calls.

### Next time ###

Next time, I'll go into detail on how we can create an asynchronous service and client, and hopefully utilize signals to add a new direction to our communication.

Again, all the code can be found on Github: [https://github.com/larryprice/python-dbus-blog-series/tree/part1](https://github.com/larryprice/python-dbus-blog-series/tree/part1).
