---
layout: post
title: "Muck with History using the HTML5 History API"
date: 2015-07-19 11:00:00 -0400
comments: true
categories: html5 web
---

Congratulations! You just made a single page web application [SPWA] to do All The Things. What happens when you hit the Back button in the browser? Oops.

The Back button is [the most used navigation button in the modern browser](http://www.internetnews.com/skerner/2010/07/what-is-the-most-clicked-firef.html). SPWA often break the functionality of the Back button. We can fix this using the [HTML 5 History API](http://diveintohtml5.info/history.html).

For this demo, I'll make a simple web page starting with the text "Page 1" and a "Continue" button. When I click the "Continue" button, "Welcome" will change to "Page 1"; when I click "Page 1", I change to "Page 2"; ad infinitum. My initial page looks like this:


``` html index.html
<span id="content" style="margin-bottom: 1.5em;">Welcome</span>
<button id="continue" onclick="nextPage()">
  Continue
</button>
```

``` javascript index.html
// ...
function nextPage() {
  var content = document.getElementById("content");
  if (content.innerHTML === "Welcome") {
    updatePageContent(1);
  } else {
    updatePageContent(parseInt(content.innerHTML.slice(5))+1);
  }
}
function updatePageContent(pageNumber) {
  document.getElementById("content").innerHTML = "Page " + pageNumber;
}
// ...
```

Running this code and clicking "Continue", the user has no way to go back. He or she would need to start over from the beginning, which is bad for the user and bad for us too. We can update the browser's history using `pushState` and `popState` commands from the History API. For this contrived example, I'll add my `pushState` call in the `nextPage` function:

``` javascript index.html
// ...
function nextPage() {
  var content = document.getElementById("content"),
      nextPage = 1;
  if (content.innerHTML === "Welcome") {
    updatePageContent(nextPage);
  } else {
    nextPage = parseInt(content.innerHTML.slice(5))+1;
    updatePageContent(nextPage);
  }

  history.pushState(null, null, "?page=" + nextPage);
}

// ...
```

Now the site will add `?page=` to the URL whenever we update the page number. This happens when we call `history.pushState`. In the call to `history.pushState`, the first argument is some data identifying the state, the second argument is the `title` of the new page but is currently unused by all the major browsers, and the third argument is the URL of the new state.

In doing this, the Back button has become enabled in the browser. However, it still doesn't work. That's where `popState` comes in. `popState` is the other side of the equation which figures out how to change the page when the user hits the Back button. I'll update the call to `pushState` to include the page number in the `state` arugment so we can use it.

``` javascript index.html
// ...

function nextPage() {
  var content = document.getElementById("content"),
      nextPage = 1;
  if (content.innerHTML === "Welcome") {
    updatePageContent(nextPage);
  } else {
    nextPage = parseInt(content.innerHTML.slice(5))+1;
    updatePageContent(nextPage);
  }

  history.pushState(nextPage, null, "?page=" + nextPage);
}

function updatePageContent(pageNumber) {
  var content = document.getElementById("content");
  if (pageNumber === 0) {
    content.innerHTML = "Welcome";
  } else {
    content.innerHTML = "Page " + pageNumber;
  }
}

window.onpopstate = function(event) {
  updatePageContent(event.state || 0);
}

// ...
```

Now your history should be _mostly_ working. Hitting Back should take you to the previous page, then hitting Forward should take you to the next page.

There is currently a bug in the version of webkit Safari uses which causes a `popstate` event to fire on initial page load. We can try to fix this using the `replaceState` call on page load to set our history's state to the initial value. You can add this event to the bottom of your javascript:

``` javascript index.html
// ... 

window.onload = function() {
  history.replaceState(0, null, "");
}

// ...
```

Of course, I want my web page to be smarter than this. If the user explicitly visits "mysweetpage.com?page=3" I want the user to be presented with page 3's epic content. Let's modify that `onload` function to check the current URL state and figure out which page the user has visited:

``` javascript index.html
// ...

window.onload = function() {
  var currentPage = parseInt(window.location.search.slice(6)) || 0;
  updatePageContent(currentPage);
  history.replaceState(currentPage, null, window.location.search);
}

// ...
```

History changed. Forever. Changing history is a good idea but can easily lead to an even more confusing user scenario. Choose when to change history based on when your application would traditionally need to bring up a new page. Once you have Back working, make sure Forward also acts as expected. Thinking about how you'll integrate the History API into your web application can help you write more sensible, modular code.

If you're looking for the full code that we created above, check out [this gist](https://gist.github.com/larryprice/b06dd00c3e7f63ae7954).

<3 HTML5
