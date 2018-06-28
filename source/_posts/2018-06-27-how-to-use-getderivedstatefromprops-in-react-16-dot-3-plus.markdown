---
layout: post
title: "How to Use getDerivedStateFromProps in React 16.3+"
date: 2018-06-27 15:12:09 -0500
comments: true
categories: react javascript
---

From [a blog post in late March 2018](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html), it was announced that the React lifecycle methods `componentWillReceiveProps`, `componentWillMount`, and `componentWillUpdate` will be deprecated in a future version of React. This is because of the eventual migration of React to async rendering; these lifecycle methods will become unreliable when async rendering is made default.

In place of these methods, the new **static** method `getDerivedStateFromProps` was introduced. My team and I struggled at first in wrapping our heads around how to migrate our many uses of `componentWillReceiveProps` to this new method. It's generally easier than you think, but you need to keep in mind that the new method is **static**, and therefore does not have access to the `this` context that the old lifecycle methods provided.

`getDerivedStateFromProps` is invoked every time a component is rendered. It takes in two arguments: the next `props` object (which may be the same as the previous object) and the previous `state` object of the component in question. When implementing this method, we need to return the changes to our component `state` or `null` (or `{}`) if no changes need to be made.

### componentWillReceiveProps ###

Here's a pattern we were using in many components throughout our codebase:

``` javascript
componentWillReceiveProps(nextProps) {
  if (nextProps.selectedTab !== this.state.selectedTab) {
    this.setState(() => { return {selectedTab: nextProps.selectedTab} })
  }
}
```

This lifecycle method fired when we were about to receive new `props` in our component, passing in the new value as the first argument. We needed to check whether the new `props` indicated a change in the state of our tab bar, which we stored in `state`. This is one of the simplest patterns to address with `getDerivedStateFromProps`:

``` javascript
static getDerivedStateFromProps(nextProps, prevState) {
  return nextProps.selectedTab === prevState.selectedTab
    ? {}
    : {selectedTab: nextProps.selectedTab}
}
```

This code works in exactly the same way, but, since it's **static**, we no longer use the context provided by `this`. Instead, we return any state changes. In this case, I've returned an empty object (`{}`) to indicate no state change when the tabs are identical; otherwise, I return an object with the new `selectedTab` value.

Sometimes you may have to perform some operations on the new `props`, but then you can still just compare the result to your previous state to figure out if anything changed. There may be other areas where you need to store some extra state duplicating your old `props` to make this work, but that may also be an indication that you need to use an alternative method.

### componentWillMount ###

We also needed to replace calls to `componentWillMount`. I found that these calls were usually directly replaceable by `componentDidMount`, which will allow your component to perform an initial render and then execute blocking tasks. This may also require adding some loading-style capacity to your component, but will be better than a hanging app.

Here's an example of a `componentWillMount` we had originally that blocked render until after an API call was made:

``` javascript
componentWillMount() {
  this.setState(() => {
    return {
      loading: 'Loading tool info'
    }
  })
  return getTool(this.props.match.params.id).then((res) => {
    this.setState(() => {
      return {
        tool: res,
        loading: null
      }
    })
  }).catch((err) => {
    api.errors.put(err)
    this.setState(() => {
      return {
        loading: null
      }
    })
  })
}
```

Afterwards, I changed the state to show the component as loading on initial render and replaced the `componentWillMount` with `componentDidMount`:

``` javascript
state = {
  tool: null,
  loading: 'Loading tool info'
}

componentDidMount() {
  return getTool(this.props.match.params.id).then((res) => {
    this.setState(() => { return {tool: res, loading: null} })
  }).catch((err) => {
    api.errors.put(err)
    this.setState(() => { return {loading: null} })
  })
}
```

### componentWillUpdate ###

Very similar to the methods discussed above, `componentWillUpdate` is invoked when a component is about to receive new props and the `render` method is definitely going to be called. Here's an example of something we were doing previously:

``` javascript
componentWillUpdate(nextProps) {
  if (!nextProps.user.isLogged && !nextProps.user.authenticating) {
    this.context.router.history.push('/')
  }
}
```

And, replacing that usage with `componentDidUpdate`:

``` javascript
componentDidUpdate(/*prevProps, prevState*/) {
  if (!this.props.user.isLogged && !this.props.user.authenticating) {
    this.context.router.history.push('/')
  }
}
```

`componentDidUpdate` is similar to `componentDidMount` except that is caused after a change in state or props occurs instead of just on initial mount. As opposed to `getDerivedStateFromProps`, we have access to the context provided by `this`. Note that this method also has arguments for `prevProps` and `prevState`, which provides the previous versions of the component's `props` and `state` for comparison to the current values.

### Conclusion ###

The deprecation of these lifecycle methods won't happen until React 17, but it's always good to plan ahead. Many of the ways my team was using these deprecated methods could be considered an anti-pattern, and I suspect that your team may be in the same predicament.
