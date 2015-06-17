---
layout: post
title: "How To ReactJS"
date: 2015-06-16 20:03:36 -0400
comments: true
categories: javascript web reactjs
---

[ReactJS](http://facebook.github.io/react/) is a component-based Javascript library for creating user interfaces. I used it extensively on recent side project [Refer Madness](https://www.refer-madness.com) ([source code](https://github.com/larryprice/refermadness)). It's pretty neat. Let's build some demos!

We'll start small. We can start with the ReactJS sample and figure out what's going on:

{% jsfiddle qqaerhp6 result,html,js light 150px %}

We create a React component `Hello` and we tell it what to render. When we create a `Hello` component, we take in an attribute called `name` which we then access in our component with `this.props.name`. When the page loads, React will render the `Hello` component in the element with ID "container".

Components can have other components.

{% jsfiddle qqaerhp6/4 result,html,js light 250px %}

I've created a `Name` component to show the name. This component uses its children to display any content it needs to. Obviously, this is a bit contrived.

Things get more exciting when we add events. Let's add an event to our `Name` component: when a user clicks this component, we'll reverse the name.

{% jsfiddle qqaerhp6/5 result,html,js light 250px %}

I've introduced state to the component. When I want a React component to use state, I need to implement the `getInitialState` method to tell React what the state is when the component is created. In this case, I set the `name` field of the component's state to `this.props.children`. When I render the component, I use the `name` variable I set in my state object. I add an `onClick` event and call a method on the component called `clickme`. In `clickme` I call the built-in `setState` method and update the `name` variable on my state object. Any other state variables on the component would remain unchanged. Any time `setState` is called, `render` is called again for this component and any children which need to be updated will also rerender.

Sometimes something happens on a child component that the parent cares about. This time, I want the parent component to know when the `Name` component gets a click event.

{% jsfiddle qqaerhp6/6 result,html,js light 250px %}

I've told my child component to call the method `goodbye` on the parent element using the property name `onClicked`. When the method is called Ichange the state of the `Hello` component and update the greeting; in this case, the `Name` component will be reused since it hasn't changed.

Just a reminder that you can have multiple of any component; just be sure to not duplicate IDs!

{% jsfiddle qqaerhp6/7 result,html,js light 250px %}

I dynamically create as many `Name` components as I need. I take my list of names and map them all to React components, each of which has a different value. As above, all of these can have events or can call events on the parent component. When we create a list of components, React prefers it when we give them all `key` attributes. This allows React to cache the components and recreate the appropriate component when states change.

Of course this is just scratching the surface. I used ReactJS because it was fast, stable, and easy to get started learning. It gave me a different way to think about the relationship between my Javascript and the DOM. For a little bit more React you should check out [the tutorials](http://facebook.github.io/react/docs/tutorial.html). If you hadn't noticed, React uses a twist on the Javascript syntax called JSX. If you're into that kind of thing, I've written about [using Docker to compile your JSX](/blog/2015/06/08/compile-jsx-with-docker/) whenever you update your JSX files - very handy IMO.
