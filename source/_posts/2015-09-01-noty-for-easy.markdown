---
layout: post
title: "Noty for Easy, Breezy, Beautiful Flash Messages"
date: 2015-09-01 18:29:27 -0400
comments: true
categories: javascript web
---

[Noty](http://ned.im/noty/#/about) is a jQuery plugin for showing flash messages. Noty is easy to use and creates great-looking flash notifications with little customization.

#### What's a flash message? ####

Flash messages are used to display small bits of information to a user without changing existing page content. If there is an error making an AJAX request, we can display an error flash message to let the user know to try again later. If there is a new shortcut available, we could let the user know with an information flash message.

#### Example Time ####

You'll need a copy of Noty. As of this time, I haven't seen a CDN hosting the minified version of Noty, so you can download it [directly from Github](https://raw.githubusercontent.com/needim/noty/master/js/noty/packaged/jquery.noty.packaged.min.js) and put it somewhere nice. Add a `<script>` tag to your HTML including the library and let's start using it.

Let's create a single button on our HTML page that, when clicked, will create a new Noty error flash message in the top left.

{% jsfiddle f14xjf8b result,html,js light 150px %}

Cool, right? Break it down:

* text
  * Obviously, this is the text that will be displayed
* layout
  * This allows us to change the position of the flash message. In this case I chose `topleft`, but other options include `top`, `topCenter`, `topRight`, `center`, `centerLeft`, `centerRight`, `bottom`, `bottomLeft`, `bottomCenter`, and `bottomRight`
* timeout
  * How long the notification will stay on the screen (in milliseconds). `false` for a permanant notification.
* type
  * Style of notification to display. Options include `error`, `success`, `alert`, `warning`, `confirmation`, and `information`.

 Let's add some more so we can see the difference.

 {% jsfiddle f14xjf8b/2 result,html,js light 150px %}

 Three notifications, one which stays up for 500ms, one which stays up for 5000ms, and one which stays up indefinitely. I also set the `killer` property to true on the `error` notification such that the `error` notification will close all other notifications. Additionally, I added the `closeWith` property to the `success` message to close it by clicking. All of these notifications come up in different locations on the page.

 We should also think about theming our notifications. To create a "full" theme for Noty is a lot of Javascript -  the odds are pretty good you just want to change the color. We can essentially inherit from the default theme and change the colors of the `error` and `information` notifictaions in our new theme like this:

``` javascript testtheme.noty.js
$.noty.themes.testtheme = Object.create($.noty.themes.defaultTheme);
$.noty.themes.testtheme.name = 'testtheme';
$.noty.themes.testtheme.style = function () {
    $.noty.themes.defaultTheme.style.apply(this);
    switch (this.options.type) {
        case 'error':
            this.$bar.css({
                backgroundColor: '#DC569F',
                borderColor: '#DC569F',
                color: '#F5F5F5'
            });
            break;
        case 'information':
            this.$bar.css({
                backgroundColor: '#ABABAB',
                borderColor: '#DC569F',
                color: '#F5F5F5'
            });
            break;
        default:
          break;
    }
    this.$message.css({
        fontWeight: 'bold'
    });
};
```

 We create our new theme `testtheme` from the `defaultTheme` prototype and change the `name` and `style` properties. Within our custom `style` function, we go ahead and apply the default theme's style. Then we add a few more adjustments based on the current notification type. Thank goodness it's javascript. Don't forget to include your `testtheme.noty.js` file in a `<script>` tag in your HTML. We can apply this theme to any of our notifications by adding the `theme` property to our call to `noty` with a value of 'testtheme' (which is the name of the theme we just created). Here's the fiddle showing off our new theme:

 {% jsfiddle f14xjf8b/3 result,html,js light 150px %}

 There are many other customization options for the brave, including custom animations and callbacks when the notifications are dismissed. See [the full documentation site](http://ned.im/noty/#/about) for more details.
