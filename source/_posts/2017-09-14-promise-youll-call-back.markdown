---
layout: post
title: "Promise you'll call back: A guide to the Javascript Promise class"
date: 2017-09-14 21:45:08 -0500
comments: true
categories: javascript nodejs
---

_This article introduces the Javascript Promise class, and how to use a Promise to perform asynchronous work. At the end of this post, you'll have been exposed to the most important components of the Promise API._

Introduced in the [ES2015 specification](http://www.ecma-international.org/ecma-262/6.0/#sec-promise-objects), [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) dryly describes a `Promise` as:

> The Promise object represents the eventual completion (or failure) of an asynchronous operation, and its resulting value.

But... what exactly does that entail? How does it differ from just using callbacks?

Let's start with a simple example. If I want to perform an operation asynchronously, traditionally I would use `setTimeout` to do work after the main thread has finished and use a callback parameter to let the caller utilize the results. For example:

``` javascript
const someAsyncTask = (after) => {
  return setTimeout(() => {
    after('the task is happening');
  }, 0);
};

console.log('before calling someAsyncTask');

someAsyncTask((result) => {
  console.log(result);
});

console.log('after calling someAsyncTask');
```

Try running this yourself with `node`, and you'll see that 'before...' and 'after...' are printed followed by 'the task is happening'.

This is perfectly valid code, but it's just so _unnatural_ to handle asynchronous tasks this way. There's no standard to which parameter should be the callback, and there's no standard to what arguments will be passed back to a given callback. Let's take a look at the same situation using the new `Promise` class:

``` javascript
const someAsyncTask = () => {
  return Promise.resolve('the task is happening');
};

console.log('before calling someAsyncTask');

someAsyncTask().then((result) => {
  console.log(result);
});

console.log('after calling someAsyncTask');
```

Let's walk through this. In `someAsyncTask`, we're now returning a call to `Promise.resolve` with our result. We call `then` on the result of `someAsyncTask` and then handle the results. `Promise.resolve` is returning a resolved `Promise`, which is run asynchronously after the main thread finishes its initial work (the final `console.log`, in this case).

Immediately, this feels a lot cleaner to me, but this is a really simple example.

Think about a situation where you need to perform multiple asynchronous callbacks that each depend on the results of the last callback. Here's an example implementation using callbacks;

``` javascript
const getFirstName = (callback) => {
  return setTimeout(() => {
    callback('Harry');
  }, 0);
};

const getLastName = (callback) => {
  return setTimeout(() => {
    callback('Potter');
  }, 0);
};

const concatName = (first, last, callback) => {
  return setTimeout(() => {
    callback(`${first} ${last}`);
  }, 0);
}

getFirstName((first) => {
  getLastName((last) => {
    concatName(first, last, (fullname) => {
      console.log(fullname);
    });
  });
});
```

I think we can all agree that this is not friendly code. What makes a `Promise` truly special is its natural chainability. As long as we keep returning `Promise` objects, we can keep calling `then` on the results:

``` javascript
const getFirstName = (callback) => {
  return Promise.resolve('Harry');
};

const getLastName = (callback) => {
  return Promise.resolve('Potter');
};

const concatName = (first, last, callback) => {
  return Promise.resolve(`${first} ${last}`);
}

getFirstName().then((first) => {
  return getLastName().then((last) => {
    return concatName(first, last);
  });
}).then((fullname) => {
  console.log(fullname);
});
```

Since `concatName` is dependent on the result of both `getFirstName` and `getLastName`, we still do a little bit of nesting. However, our final asynchronous action can now occur on the outside of the nesting, which will take advantage of the last returned result of our `Promise` resolutions.

Error handling is another can of worms in callbacks. Which return value is the error and which is the result? Every level of nesting in a callback has to either handle errors, or maybe the top-most callback has to contain a try-catch block. Here's a particularly nasty example:

``` javascript
const getFirstName = (callback) => {
  return setTimeout(() => {
    if (Math.random() > 0.5) return callback('Sorry, firstName errored');
    callback(null, 'Harry');
  }, 0);
};

const getLastName = (callback) => {
  return setTimeout(() => {
    if (Math.random() > 0.5) return callback('Sorry, lastName errored');
    callback(null, 'Potter');
  }, 0);
};

const concatName = (first, last, callback) => {
  return setTimeout(() => {
    if (Math.random() > 0.5) return callback('Sorry, fullName errored');
    callback(null, `${first} ${last}`);
  }, 0);
}

getFirstName((err, first) => {
  if (err) console.error(err); // no return, will fall through despite error
  getLastName((err, last) => {
    if (err) return console.error(err);
    concatName(first, last, (err, fullname) => {
      if (err) return console.error(err);
      console.log(fullname);
    });
  });
});
```

Every callback has to check for an individual error, and if any level mishandles the error (note the lack of a return on error after `getFirstName`), you're guaranteed to end up with undefined behavior. A `Promise` allows us to handle errors at any level with a `catch` statement:

``` javascript
const getFirstName = () => {
  if (Math.random() > 0.5) return Promise.reject('Sorry, firstName errored');
  return Promise.resolve('Harry');
};

const getLastName = () => {
  if (Math.random() > 0.5) return Promise.reject('Sorry, lastName errored');
  return Promise.resolve('Potter');
};

const concatName = (first, last) => {
  if (Math.random() > 0.5) return Promise.reject('Sorry, fullName errored');
  return Promise.resolve(`${first} ${last}`);
}

getFirstName().then((first) => {
  return getLastName().then((last) => {
    return concatName(first, last);
  });
}).then((fullname) => {
  console.log(fullname);
}).catch((err) => {
  console.error(err);
});
```

We return the result of `Promise.reject` to signify that we have an error. We only need to call `catch` once. Any `then` statements from unresolved promises will be ignored. A `catch` could be inserted at any nesting point, which could give you the ability to continue the chain:

``` javascript
// ...

getFirstName().then((first) => {
  return getLastName().then((last) => {
    return concatName(first, last);
  }).catch((err) => {
    return concatName(first, 'Houdini');
  });
}).then((fullname) => {
  console.log(fullname);
}).catch((err) => {
  console.error(err);
});
```

So far, we've been returning `Promise` objects using `resolve` and `reject`, but there's also the ability to define our own `Promise` objects with their own `resolve` and `reject` methods. Updating the `getFirstName` variable:

``` javascript
const getFirstName = () => {
  return new Promise((resolve, reject) => {
    if (Math.random() > 0.5) return reject('Sorry, firstName errored');
    return resolve('Harry');
  });
}
// ...
```

We can also run our asynchronous tasks without nesting by using the `Promise.all` method:

``` javascript
// ...

Promise.all([getFirstName(), getLastName()]).then((names) => {
  return concatName(names[0], names[1]);
}).then((fullname) => {
  console.log(fullname);
}).catch((err) => {
  console.error(err);
});
```

Give `Promise.all` a list of promises and it will call them (in some order) and return all the results in an array (in the order given) as a resolved `Promise` once all given promises have been resolved. If any of the promises are rejected, the entire `Promise` will be rejected, resulting in the `catch` statement.

Sometimes you need to run several methods, and you only care about the first result. `Promise.race` is similar to `Promise.all`, but only waits for one of the given promises to return:

``` javascript
const func1 = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve('func1'), 5*Math.random());
  });
}

const func2 = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve('func2'), Math.random());
  });
}

Promise.race([func1(), func2()]).then((name) => {
  console.log(name);
}).catch((err) => {
  console.error(err);
});
```

Sometimes, 'func1' will be printed, but most of the time 'func2' will be printed.

...And that's the basics! Hopefully, you have a better understanding of how a `Promise` works and the advantages provided over traditional callback architectures. More and more libraries are depending on the `Promise` class, and you can really clean up your logic by using those methods. As Javascript continues to evolve, hopefully we find ourselves getting more of these well-designed systems to make writing code more pleasant.
