---
layout: post
title: "Getting Started with React Hooks"
date: 2019-02-09 13:10:46 -0600
comments: true
categories: reactjs javascript
---

[React Hooks][hooks-guide] are a new feature allowing developers to use state in functional components officially released in [React 16.8][react-16-8]. I am in love with the idea of Hooks, so much so that I'm giving an introductory talk on the topic at an [upcoming local JavaScript meetup][meetup].

All code from this post can be found in a [codepen collection][codepen].

### Functional Components ###

A functional component (also sometimes referred to as a "stateless" component) is a method of defining React components with only a render method. These components still take in readonly props and return some JSX, but until now have had no means to perform any stateful logic. The following is a simple functional component that creates a button for canceling a user's account:

``` javascript https://codepen.io/larryprice/pen/BMJYwe
const CancelAccountDeletion = ({onClick}) => (
  <button className="btn btn-default btn-lg cancel-account-deletion" onClick={onClick}>
    <span className="glyphicon glyphicon glyphicon-ban-circle"></span>
    Cancel Account
  </button>
);
```

The component takes in a single prop `onClick` that is called when the button is clicked.

### Adding Stateful Logic The Old Way ###

Let's say that we want to add some stateful logic to that component. Marketing has started complaining that users clicking our current "Cancel Account" button contribute to a loss of revenue, and we need to slow that loss down to appease investors this quarter. We get design involved and decide to prompt the user several times to confirm their cancellation. We'll need to keep track of the number of clicks and the current prompt in state.

Here's how we might do that on February 5th, 2019, using class components:

``` javascript https://codepen.io/larryprice/pen/XOgKYd
const messages = ['Really?', 'Don\'t leave me!', 'OK, fine!'];
class CancelAccountDeletion extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      buttonText: 'Cancel',
      clicks: 0,
    }
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.clicks !== this.state.clicks) {
      if (this.state.clicks < messages.length) {
        return this.setState((prevState) => ({
          buttonText: messages[prevState.clicks],
        }));
      }

      return this.props.onClick();
    }
  }

  render() {
    return (
      <button className="btn btn-default btn-lg cancel-account-deletion"
          onClick={() => this.setState((prevState) => ({
            clicks: prevState.clicks + 1,
          }))}>
        <span className="glyphicon glyphicon glyphicon-ban-circle"></span>
        {this.state.buttonText}
      </button>
    );
  }
}
```

Wow! We've nearly tripled the size of our original component here. In this stateful world, we needed to extend the `React.Component` class, define a constructor to set our initial state, update our state when the button is clicked, and add the `componentDidUpdate` lifecycle method. The `componentDidUpdate` method is called on every re-render, so we first check to see if the number of `clicks` changed before taking any action. If it did, we check to see if we have more messages than clicks and update the prompt text; otherwise, we call the original `onClick` function from our props and, unfortunately for our sales goals, churn another user.

This is a lot of boilerplate and has a tendency to get complex really fast. If only there was another way!

"Well, actually, Papa Larry," I hear you interjecting from behind your monitor, "we could do this without a lifecycle method and only one piece of state." My dear friend. Yes, this code is slightly contrived so that I can show you all the main features of hooks with a fairly straightforward example. Just keep your susurruses to yourself until after the show.

### Adding Stateful Logic the New Way ###

This is where Hooks come into play. Let's fast-forward from early evening in the American Midwest on February 5th, 2019, to late evening, when suddenly React 16.8 was released and it was officially titled "The One With Hooks."

Let's take our original functional component and add state with Hooks:

``` javascript https://codepen.io/larryprice/pen/vbgwGx
const messages = ['Cancel', 'Really?', 'Don\'t leave me!', 'OK, fine!'];
const CancelAccountDeletion = ({onClick}) => {
  const [clicks, setClicks] = useState(0);
  const [buttonText, setButtonText] = useState();

  useEffect(() => {
    if (clicks < messages.length) {
      return setButtonText(messages[clicks]);
    }
    return onClick();
  }, [clicks]);

  return (
    <button onClick={() => setClicks(clicks+1)}
        className="btn btn-default btn-lg cancel-account-deletion">
      <span className="glyphicon glyphicon glyphicon-ban-circle"></span>
      {buttonText}
    </button>
  );
};
```

Our Hooks implementation is about half as long as our class implementation. I would argue that it's also significantly easier to read. Let's break this down bit-by-bit to discuss each piece of the hooks API:

``` javascript
const [clicks, setClicks] = useState(0);
const [buttonText, setButtonText] = useState();
```

At the top of our function, we call the `useState` method to declare two state variables: `clicks` and `buttonText`. `useState` takes in an initial value and returns a state variable and setter method, which we access locally using array destructuring. In this case, we set the initial state of `clicks` to `0` and leave `buttonText` empty.

Behind-the-scenes, React is using our component's scope to create and track these state variables. We _must_ always define these variables in the same order when this function is executed, or we'll get our variables all mixed up and our logic won't make any sense.

``` javascript
useEffect(() => {
  if (clicks < messages.length) {
    return setButtonText(messages[clicks]);
  }
  return onClick();
}, [clicks]);
```

The `useEffect` method is essentially a replacement for the `componentDidMount` and `componentDidUpdate` lifecycle methods. It takes in a function that will be called after every render. Here we take advantage of closures to test the value of our `clicks` state variable and use `setButtonText` to update our `buttonText` state variable. The second argument to `useEffect` is an array of state variables to check - if none of the given state variables were changed, the effect will be skipped.

We can call `useEffect` as many times as we want in our component. This allows us to create a clear separation of concerns if we need to define several different effects.

``` javascript
return (
  <button onClick={() => setClicks(clicks+1)}
      className="btn btn-default btn-lg cancel-account-deletion">
    <span className="glyphicon glyphicon glyphicon-ban-circle"></span>
    {buttonText}
  </button>
);
```

This is our same old render logic, but in this case we're using the `setClicks` function returned to us by `useState`.

### Custom Hooks ###

Design and marketing like this concept of delaying an action and just changing the text so much that they want to use it all over the site. Now we have stateful logic that needs to be reused. This is where the concept of "Custom Hooks" comes in:

``` javascript https://codepen.io/larryprice/pen/GzENrZ
const useTextByCount = (count, messages, onFinished) => {
  const [text, setText] = useState(messages[0]);

  useEffect(() => {
    if (count < messages.length) {
      return setText(messages[count]);
    }
    return onFinished();
  }, [count]);

  return text;
};

const messages = ['Cancel', 'Really?', 'Don\'t leave me!', 'OK, fine!']
const CancelAccountDeletion = ({onClick}) => {
  const [clicks, setClicks] = useState(0);
  const buttonText = useTextByCount(clicks, messages, onClick);

  return (
    <button onClick={() => setClicks(clicks+1)}
        className="btn btn-default btn-lg cancel-account-deletion">
      <span className="glyphicon glyphicon glyphicon-ban-circle"></span>
      {buttonText}
    </button>
  )
};
```

Here I've created my own hook called `useTextByCount` that abstracts away the entire concept of the `buttonText` state variable. We can use this custom hook in any functional component. Abstracting stateful logic is a tall task in class components, but it's completely natural using Hooks.

### Conclusion ###

Hooks are the result of the React maintainers responding to the way React developers want to write code, enabling us to use powerful stateful concepts in a cleaner, functional system. This is a natural next step for the React API, but it's not going to deprecate all your class components. Hooks are completely optional and backwards compatible with current React concepts, so there's no need to make a Jira ticket to refactor all your components tomorrow morning. Hooks are here to help you write new components faster and better, giving you new options when you need to start adding state to that simple button component.

Check out the [Hooks Guide][hooks-guide] and the [Rules of Hooks][hooks-rules] for more information.

Happy hooking!

[hooks-guide]: https://reactjs.org/docs/hooks-intro.html
[react-16-8]: https://reactjs.org/blog/2019/02/06/react-v16.8.0.html
[hooks-rules]: https://reactjs.org/docs/hooks-rules.html
[abramov]: https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889
[meetup]: https://www.meetup.com/C-U-JavaScript/events/258294308/
[codepen]: https://codepen.io/collection/XMoJzy/
