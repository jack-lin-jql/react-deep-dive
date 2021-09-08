# Using the effect hook

- Effect hooks lefts one perform side effects in function components

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <p>You clicked {count} times</p>
    <button onClick={() => setCount(count + 1)}>
      Click me
    </button>
  );
}
```

- Data fetching, setting up a subscription and manually changing the DOM in React components are all examples of side effects
- Think of the `useEffect` hook as `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` combined

## Effects without cleanup

- At times, one needs to run some additional code after React has updated the DOM
  - Network requests, manual DOM mutations, and logging are common examples of effects that don't require a cleanup, aka, run them and immediately forget about them
  
### Example using classes

- In React class components, `render` itself shouldn't cause side effects, it would be too early, we usually perform effects after React has updated the DOM

```js
class Example extends React.Components {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

- Note the duplicated code between the two lifecycle method in class
  - This is because in many cases, the same side effect is performed regardless of whether the component just mounted or it had been updated
  - Conceptually, we want it to happen after every render, but React class components don't have a method like this

### Example using hooks

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

- What does useEffect do? - Tells React that the component needs to do something after render
  - React will remember the function passed and call it later after performing the DOM updates
  - One can also perform data fetching or call some other imperative API within the effect
- Why is useEffect called inside a component? - Allows access to the `count` state variable (or any props) right from the effect
  - Hooks embrace JS closures and avoid introducing React-specific APIs when JS already provides a solution
- Does useEffect run after every render? - Yes, by default, it runs both after the first render and after every update
  - Instead of thinking in terms of mounting and updating, one might find it easier to think that effects happen "after render"
  - React guarantees the DOM has been updated by the time it runs the effects

### Details explanation

```
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
}
```

- Here `count` is declared, an effect function is passed to the `useEffect` hook
- The effect function sets the document title
- Latest `count` can be read inside the effect since it's in the scope of this function
- When React renders the component, it'll remember that effect used and run the effect after updating the DOM, which happens for every render, including the first one
- The function passed into `useEffect` is going to be different on every render which is intentional
  - This is what lets reading the `count` value from inside the effect without worrying about it getting stale
  - Every time re-render takes place, a different effect is scheduled, replacing the previous one
  - This makes the effect behave more like a part of the render result - each effect belongs to a particular render

### Tip

- Unlike `componentDidMount` or `componentDidUpdate`, effects scheduled with `useEffect` don't block the browser from updating the screen
- This makes apps feel more responsive and majority of the effects don't need to happen synchronously
- In uncommon cases where they do (such as measuring the layout), use `useLayoutEffect` hook instead

## Effects with cleanup

- Some effects requires cleanup if they, for example, deal with subscription to some external data source. It is important to clean up to ensure no memory leak

### Example using classes

```js
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }

    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```

- Note how `componentDidMount` and `componentWillUnmount` mirrors each other
- Lifecycle methods forces the split of this logic even though conceptually code in both of them is related to the same effect

### Example using hooks

- If an effect returns a function, React will run it when it is time to clean up

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }

  return isOnline ? 'Online' : 'Offline';
}
```

- Why did we return a function from our effect? - It is an optional cleanup mechanism for effects
  - Every effect may return a function that cleans up after it
  - Allows the logic for adding and removing subscription close to each other as they're part of the same effect
- When exactly does React clean up an effect? - React performs the cleanup when the component unmounts
  - However! Effects run for every render and not just once
  - This is why React also cleans up effects from the previous render before running the effects next time

## Tip: Use multiple effects to separate concerns

- A problem with class lifecycle methods is that they often contain unrelated logic, but related logic gets broken up into several methods

```js
class FriendStatusWithCount extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null};
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
}
```

- Note the logic that sets `document.title` is split between `componentDidMount` and `componentDidUpdate`
  - The subscription logic is also split between `componentDidMount` and `componentWillUnmount`
  - `componentDidMount` contains code for both tasks
- We can improve this by using multiple effects in a function component

```js
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

- Hooks lets us split the code based on what it is doing rather than a lifecycle method name
- React will apply every effect used by the component, in the order they were specified