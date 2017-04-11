---
layout: post
title: "Using D-Bus Signals in Python"
date: 2017-04-11 15:30:58 -0400
comments: true
categories: python dbus ubuntu
---

_This is the third in a series of blog posts on creating an asynchronous D-Bus service in python. For the inital entry, [go here](/blog/2017/04/04/creating-a-d-bus-service-with-python/). For the previous entry, [go here](/blog/2017/04/05/creating-an-asynchronous-d-bus-service-with-python/)_

Last time we transformed our base synchronous D-Bus service to include asynchronous calls in a rather naive way. In this post, we'll refactor those asynchronous calls to include D-Bus signals; codewise, we'll pick up right where we left off after part 2: [https://github.com/larryprice/python-dbus-blog-series/tree/part2](https://github.com/larryprice/python-dbus-blog-series/tree/part2). Of course, all of today's code can be found in the same project with the part3 tag: [https://github.com/larryprice/python-dbus-blog-series/tree/part3](https://github.com/larryprice/python-dbus-blog-series/tree/part3).

### Sending Signals ###

We can fire signals from within our D-Bus service to notify clients of tasks finishing, progress updates, or data availability. Clients subscribe to these signals and act accordingly. Let's start by changing the signature of the `slow_result` method of `RandomData` to be a signal:

``` python random_data.py
# ...
@dbus.service.signal("com.larry_price.test.RandomData", signature='ss')
def slow_result(self, thread_id, result):
    pass
```

We've replaced the context decorator with a `signal`, and we've swapped out the guts of this method for a `pass`, meaning the method will call but doesn't do anything else. We now need a way to call this signal, which we can do from the `SlowThread` class we were using before. When creating a `SlowThread` in the `slow` method, we can pass in this signal as a callback. At the same time, we can remove the `threads` list we used to use to keep track of existing `SlowThread` objects.

``` python random_data.py
class RandomData(dbus.service.Object):
    def __init__(self, bus_name):
        super().__init__(bus_name, "/com/larry_price/test/RandomData")

        random.seed()

    @dbus.service.method("com.larry_price.test.RandomData",
                         in_signature='i', out_signature='s')
    def slow(self, bits=8):
        thread = SlowThread(bits, self.slow_result)
        return thread.thread_id

    # ...
```

Now we can make some updates to `SlowThread`. The first thing we should do is add a new parameter `callback` and store it on the object. Because `slow_result` no longer checks the `done` property, we can remove that and the `finished` event. Instead of calling `set` on the event, we can now simply call the `callback` we stored with the current `thread_id` and `result`. We end up with a couple of unused variables here, so I've also gone ahead and refactored the `work` method on `SlowThread` to be a little cleaner.

``` python
# ...

class SlowThread(object):
    def __init__(self, bits, callback):
        self._callback = callback
        self.result = ''

        self.thread = threading.Thread(target=self.work, args=(bits,))
        self.thread.start()
        self.thread_id = str(self.thread.ident)

    def work(self, bits):
        num = ''

        while True:
            num += str(random.randint(0, 1))
            bits -= 1
            time.sleep(1)

            if bits <= 0:
                break

        self._callback(self.thread_id, str(int(num, 2)))
```

And that's it for the service-side. Any callers will need to subscribe to our `slow_result` method, call our `slow` method, and wait for the result to come in.

### Receiving Signals ###

We need to make some major changes to our `client` program in order to receive signals. We'll need to introduce a main loop, which we'll spin up in a separate thread, for communicating on the bus. The way I like to do this is with a ContextManager so we can guarantee that the loop will be exited when the program exits. We'll move the logic we previously used in `client` to get the `RandomData` object into a private member method called `_setup_object`, which we'll call on context entry after creating the loop. On context exit, we'll simply call `quit` on the loop.

``` python client
# Encapsulate calling the RandomData object on the session bus with a main loop
import dbus, dbus.exceptions, dbus.mainloop.glib
import threading
from gi.repository import GLib
class RandomDataClient(object):
    def __enter__(self):
        self._setup_dbus_loop()
        self._setup_object()

        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self._loop.quit()
        return True

    def _setup_dbus_loop(self):
        dbus.mainloop.glib.DBusGMainLoop(set_as_default=True)
        self._loop = GLib.MainLoop()

        self._thread = threading.Thread(target=self._loop.run)
        self._thread.start()

    def _setup_object(self):
        try:
            self._bus = dbus.SessionBus()
            self._random_data = self._bus.get_object("com.larry-price.test",
                                                     "/com/larry_price/test/RandomData")
        except dbus.exceptions.DBusException as e:
            print("Failed to initialize D-Bus object: '%s'" % str(e))
            sys.exit(2)
```

We can add methods on `RandomDataClient` to encapsulate `quick` and `slow`. `quick` is easy - we'll just return `self._random_data.quick(bits)`. `slow`, on the other hand, will take a bit of effort. We'll need to subscribe to the `slow_result` signal, giving a callback for when the signal is received. Since we want to wait for the result here, we'll create a `threading.Event` object and `wait` for it to be `set`, which we'll do in our handler. The handler, which we'll call `_finished` will validate that it has received the right result based on the current `thread_id` and then set the `result` on the `RandomDataClient` object. After all this, we'll remove the signal listener from our bus connection and return the final result.

``` python client
class RandomDataClient(object):
    # ...

    def quick(self, bits):
        return self._random_data.quick(bits)

    def _finished(self, thread_id, result):
        if self._thread_id == self._thread_id:
            self._result = result
            self._done.set()

    def slow(self, bits):
        self._done = threading.Event()
        self._thread_id = None
        self._result = None

        signal = self._bus.add_signal_receiver(path="/com/larry_price/test/RandomData", handler_function=self._finished,
                                               dbus_interface="com.larry_price.test.RandomData", signal_name='slow_result')
        self._thread_id = self._random_data.slow(bits)
        self._done.wait()
        signal.remove()

        return self._result
```

Now we're ready to actually call these methods. We'll wrap our old calling code with the `RandomDataClient` context manager, and we'll directly call the methods as we did before on the client:

``` python client
# ...

# Call the appropriate method with the given number of bits
with RandomDataClient() as client:
    if args.slow:
        print("Your random number is: %s" % client.slow(int(args.bits)))
    else:
        print("Your random number is: %s" % client.quick(int(args.bits)))
```

This should have feature-parity with our part 2 code, but now we don't have to deal with an infinite loop waiting for the service to return.

### Next time ###

We have a working asynchronous D-Bus service using signals. Next time I'd like to dive into forwarding command output from a D-Bus service to a client.

As a reminder, the end result of our code in this post is MIT Licensed and can be found on Github: [https://github.com/larryprice/python-dbus-blog-series/tree/part3](https://github.com/larryprice/python-dbus-blog-series/tree/part3).
