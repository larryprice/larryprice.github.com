---
layout: post
title: "I am an Artist Now: CSS3 Transitions"
date: 2015-04-06 18:57:36 -0400
comments: true
categories: css
---

I just discovered [CSS3 Animation](//www.w3.org/TR/css3-animations/) and I am quickly becoming obsessed.

<a name="infinite-ball"></a>
{% jsfiddle 9uny3mb1/1 result,html,css light 150px %}

CSS3 makes DOM animations really easy. I can move things around, shrink or enlarge text and images, make buttons and links "pop" on hover, and help the user focus on the most relevant information. This can all be done through CSS3.

__DISCLAIMER__: Contrary to the title of this article, I am less of a Picasso and more of a dog making "art" all over your brand new rug.

### Animation ###

Do you see that ball [up there](#infinite-ball)? If you can't, go check out the [jsfiddle](//jsfiddle.net/9uny3mb1/1). The concepts are so simple an embedded programmer could do it. Let's look at the `animation-` tags on the ball first:

``` css
.ball {
  /* ... */
  animation-duration: 10s;
  animation-name: rollout;
  animation-iteration-count: infinite;
}
```

* `animation-duration`
  * The length of time the animation will take to complete.
  * Example: 10s
* `animation-name`
  * A unique name for the animation we want to use. If you write good animations, you might be able to reuse them later.
  * Example: rollout
* `animation-iteration-count`
  * The number of times the animation should run. Default 1.
  * Example: infinite

Note that there are several other `animation-` attributes to give you more control, but we only need a few in this case. For a more in-depth look at these attributes and their constituents, I advise you to check out [this MDN article](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Using_CSS_animations).

The `animation-` attributes let us configure our usage of an animation, but the real guts of the animation come from the definition inside a `@keyframes` tag.

``` css
.ball { /* ... */ }

@keyframes rollout {
  25% {
    margin-left: 47%;
  }
  50% {
    margin-left: 94%;
  }
  75% {
    margin-left: 47%;
  }
  100% {
    margin-left: 0%;
  }
}
```

Note the name `rollout` is the same as the `animation-name` attribute we set on `.ball` previously. This is the definition of our animation at different timeframes. What this says:

1. __25%__ of the way through the animation, the object should have `margin-left` of __47%__, putting it in the middle of the page.
1. __50%__ of the way through the animation, the object should have `margin-left` of __94%__, putting it at the far right of the page.
1. __75%__ of the way through the animation, the object should have `margin-left` of __47%__, putting it in the middle of the page again.
1. __100%__ of the way through the animation, the object should have `margin-left` of __0%__, putting it back where it started.

This is easy and relatively readable. I can see exactly what is supposed to be going on and when.

### Transition ###

Although the `animation-` attributes give you a lot of control over animations, there are several other tools available. [CSS Transitions](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Using_CSS_transitions) are animations that are performed [between two distinct states](//stackoverflow.com/questions/20586143/css-animation-vs-transition#20590319). This is especially helpful when you want to run an animation after some event happens. When I click this button, I add a class to my ball to make it move to the right. I click it again and the ball moves to the left. You can see [the fiddle below](#ball-click) or view it on [jsfiddle](//jsfiddle.net/9uny3mb1/4/).

<a name="ball-click"></a>
{% jsfiddle 9uny3mb1/4 result,html,css,js light 150px %}

The important CSS here is `transition` and the `margin-left`:

``` css
.ball {
  /* ... */

  transition: margin-left 3s;   
  margin-left: 0%;
}

.rollout {
  margin-left: 94%;
}
```

When my DOM object has the `ball` class, a transition will begin to set the `margin-left` to __0%__ over a period of 3 seconds. When the `rollout` class is applied, a transition begins which brings the `margin-left` to __94%__ over another 3 seconds. The transition knows to apply the given CSS attribute as found in the last-applicable style definition.

Transitions are really great for single page applications - as I switch contexts in my application, I move my DOM elements around or I gradually hide them from view. When I get time to release my next homemade application, expect to see lots of little animations. Probably way more animations than necessary - just bear with me.

There are many other CSS animation topics, but I don't quite have the experience to elaborate. [Transforms](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Using_CSS_transforms) give you really easy ways to rotate, skew, and translate objects, even allowing for the creation of 3-dimensional CSS objects. The `to` and `from` shorthand in `@keyframes` elements can make reading animations even easier. I may be a little late to the party, but the world created by CSS3 and HTML5 is a much better one for programmers and (hopefully) for end-users.
