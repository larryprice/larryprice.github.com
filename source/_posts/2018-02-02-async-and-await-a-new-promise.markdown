---
layout: post
title: "Async and Await - A New Promise"
date: 2018-02-02 15:19:48 -0600
comments: true
categories: javascript nodejs
---

In my [last post](/blog/2017/09/14/promise-youll-call-back/), I discussed the [ES2015 concept of a `Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise). A `Promise` provides a simplified mechanism for performing asynchronous work in JavaScript without using the classic `setTimeout`-callback approach. Seeing as it's been about 4 months since my previous post, a new asynchronous concept is on the rise as part of the [ES2017 specification](https://tc39.github.io/ecma262/2017/#sec-async-function-definitions): [`async` and `await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).

I became aware of `async` and `await` after reading [David Walsh's blog](https://davidwalsh.name/async-await), at which point I disregarded the new features as being "too soon" and "not different enough" from a `Promise` to warrant a second thought. Then, yesterday, I used them, and my life was, once again, forever changed.

`await` is used to essentially wait for a `Promise` to finish. Instead of using a callback with a `then` clause, `await` allows you to perform the action and/or store the result like you're within a synchronous function.

`async` is a keyword identifier used on functions to specify that that function will use `await`. Try to call `await` in a function not labeled as `async` and you're going to have a bad time. Any `async` function returns a `Promise`.

Let's see an example:

``` javascript
function getFirstName() { return Promise.resolve('Charles'); }
function getMiddleName() { return Promise.resolve('Entertainment'); }
function getLastName() { return Promise.resolve('Cheese'); }

async function getName() {
  const first = await getFirstName();
  const middle = await getMiddleName();
  const last = await getLastName();

  return `${first} ${middle} ${last}`;
}

getName().then((name) => {
  console.log(name);
});

console.log('My next guest needs no introduction:');

// Result:
//   My next guest needs no introduction:
//   Charles Entertainment Cheese
```

We have three functions which each return a `Promise`, and an `async` function which calls those functions sequentially and uses the results to construct a string. We call the `getName` function (which is `async` and therefore returns a `Promise`) and log the results. Our last command logs a special message. Due to the asynchronous nature of the `getName` function, our special message is logged first, and then the result of `getName`.

This comes in handy when you're depending on the results of a `Promise` to do some work or pass into another asynchronous call. But, in the case of our `getName` function above, we could be getting all three of the names at once. This calls for the brilliant `Promise.all` method, which can also be used with `async`. Let's modify our sub-name functions to all use `async` and then fetch them all at once:

``` javascript
async function getFirstName() { return 'Charles'; }
async function getMiddleName() { return 'Entertainment'; }
async function getLastName() { return 'Cheese'; }

async function getName() {
  const names = await Promise.all([getFirstName(), getMiddleName(), getLastName()]);

  return `${names[0]} ${names[1]} ${names[2]}`;
}

getName().then((name) => {
  console.log(name);
});

console.log('My next guest needs no introduction:');

// Result:
//   My next guest needs no introduction:
//   Charles Entertainment Cheese
```

Since an `async` function just returns a `Promise`, we can directly use (and even inter-mix) `async` functions inside `Promise.all`, and the results come back in an ordered array.

OK, what if we want to fire off some long-running task and do some other work in the meantime? We can defer our use of `await` until after we've performed all the intermediate work:

``` javascript
async function getFirstName() { return 'Charles'; }
async function getMiddleName() { return 'Entertainment'; }
async function getLastName() { return 'Cheese'; }

async function getName() {
  const first  = getFirstName();  // first, middle, and last will all
  const middle = getMiddleName(); // be pending Promises at this
  const last   = getLastName();   // point, to be resolved in time

  const title = Math.random() > .5 ? 'Sr.' : 'Esq.';

  return `${await first} ${await middle} ${await last}, ${title}`;
}

getName().then((name) => {
  console.log(name);
});

console.log('My next guest needs no introduction:');

// Result will be quasi-random:
//   My next guest needs no introduction:
//   Charles Entertainment Cheese, (Esq.|Sr.)
```

This example reiterates that you can use `async` functions just like you would a `Promise`, but with the added benefit of using `await` to wait for the results when necessary.

I know what you're thinking: "All these positives, Larry! Is there nothing negative about `async`/`await`?" As always, there are a couple of pitfalls to using these functions. The biggest nuisance for me is the loss of the `catch` block when converting from a `Promise` chain. In order to catch errors with `async`/`await`, you'll have to go back to traditional `try/catch` statements:

``` javascript
async function checkStatus() { throw 'The Cheese is displeased!'; }

async function checks() {
  try {
    await checkStatus();
    return 'No problems.';
  } catch (e) {
    return e;
  }
}

checks().then((status) => {
  console.log(status)
})
console.log('Current status:');

// Result will be quasi-random:
//   Current status:
//   The Cheese is displeased!
```

The only other real downside is that `async` and `await` may not be fully supported in your users' browsers or your version of Node.JS. There are plenty of ways to get around this with Babel and polyfills, but, to be honest, I dedicated a large chunk of time yesterday afternoon to upgrading all of our libraries and babel versions to get this to work properly everywhere. Your mileage may vary, and, if you're reading this 6 months from when it was posted, I'm sure it will be available by default in any implementations of ECMAScript.
