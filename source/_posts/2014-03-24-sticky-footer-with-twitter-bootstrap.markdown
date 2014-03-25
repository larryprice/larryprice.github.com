---
layout: post
title: "Sticky footer with Twitter Bootstrap"
date: 2014-03-24 20:12:50 -0400
comments: true
categories: twitter-bootstrap css html ollert
---

Sometimes CSS is a total pain.

We encountered a major CSS problem while working on our incredible startup weekend project [Ollert](//ollert.herokuapp.com). We had created a footer that we wanted below all of our content. We threw together some quick HTML and got a footer below all of the main content, and it looked really good when our main content filled up the entire screen.

What about when there was very little data on the screen? Well, then the footer just floated in the middle of the page, staring at us like some kind of psychotic hummingbird, waiting to slice you up when you're not looking. We searched online and found lots of different solutions; None of them worked. The footer just floated there, taunting us; Telling us to cry home to mommy. So we gave up on the prospect for the rest of the afternoon.

A few days after startup weekend, I found the real solution from the good folks at [Twitter Bootstrap](//getbootstrap.com/2.3.2/examples/sticky-footer.html) themselves. It's pretty simple, really. Hooray for the internet!

Below is the HTML to create this effect with all the CSS styles embedded. Marked up with plenty of comments.

``` html index.html
<!DOCTYPE html>
<html lang="en" style="height: 100%;">
  <head>
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.1.1/css/bootstrap.min.css">
  </head>
  <body style="height: 100%;">
    <div id="wrap" style="min-height: 100%; height: auto !important; height: 100%; margin: 0 auto -50px;">
      <!-- All your content goes inside wrap. -->
      <!-- The bottom margin must be negative the footer min-height -->
      <!-- Footer min-height is set to 50px in this case -->
      <div class="container">
        <div class="row">
          <div class="h1">
            All Your Content
          </div>
        </div>
        <div class="row">
          All your content will go inside the 'wrap' div.
        </div>
      </div>
      <div id="push" style="min-height: 50px;">
        <!-- This push node should be inside wrap and empty -->
        <!-- Min-height is equal to the min-height of the footer -->
      </div>
    </div>
  </body>
  <div id="footer" style="min-height: 50px;">
    <!-- Some sweet footer content -->
    <div class="container">
      <div class="small">
        Zowee, I've got a footer stuck to the bottom!
      </div>
    </div>
  </div>
</html>

```

Div tag ids such as "wrap", "push", and "footer" can be whatever you want. The height of the footer can be adjusted to fit whatever content you want; I found that using `min-height` instead of `height` allowed my content to resize appropriately when wrapped. Styles should definitely be moved to a css file.
