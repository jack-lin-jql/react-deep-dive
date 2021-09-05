# Hooks at a glance

- Lets one use state and other React features without writing a class
- Backwards compatible

## State hook

```javascript
import React, { useState } from "react";

function Example() {
  // Declare a new state variable
  const [count, setCount] = useState(0);

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

- `useState` is a hook that's called inside a function to add some local state
- React preserves this state between ren-renders
- `useState` returns a pair: the current state value and a function that lets one update it, similar to `this.setState`, but it doesn't merge old and new state together
- `useState` accepts an initial state

### Declaring multiple state variables

```js
function ExampleWithManyStates() {
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState[{ text: 'Learn hooks' }];
  // ..
}
```

- The array destructuring syntax lets one give different names to the state variables by calling `useState`
- React assumes that if one calls `useState` many times, one does it in the same order during every render

### But what is a Hook?

- Hooks are functions that lets one "hook into" React state and lifecycle features from function components
- Hooks don't work inside classes as they let one use React without classes

## Effect hook

- Performing data fetching, subscription, or manually changing the DOM from React component are called side effects ("effects" for short) since they can affect other components and can't be done during rendering
- `useEffect` adds the ability to perform side effects from a function component
- Serves the same purpose as `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`, but unified into a single API
- E.g. the following sets the document title after React updates the DOM:

```js
import React, { useState, useEffect } from "react";

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser APIP
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

- When `useEffect` is called, it is telling React to run the "effect" function **after flushing** changes to the DOM
- They are declared inside the component so they have access to props and states
- By default, React runs the effects after every render, including the first render
- Effects can specify how to clean up after them by returning a function

```js
import React, { useState, useEffect } from "react";

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if(isOnline === null) {
    return "Loading...";
  }

  return isOnline ? "Online" : "Offline";
}
```

- Here, React would unsubscribe from `ChatAPI` when component unmounts, as well as before re-running the effect due to a subsequent render
- `useEffect` can be used multiple times in a component

```js
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  //...
}
```

- Hooks lets one organize side effects in a component by what pieces are related rather than forcing a split based on lifecycle methods

## Rules of hooks

- Hooks are JS functions but they impose 2 additional rules
  - Only call hooks at the top level. Don't call hooks inside loops, conditions, or nested functions
  - Only call hooks from React function components. Don't call hooks from regular JS functions

## Building one's own hooks

- To reuse stateful logic between components
- Traditionally, this was done using HOCs or render props
- Custom hooks lets one do this without adding more components to the tree
- Let's reuse the subscription logic for `FriendStatus` and call it `useFriendStatus`

```js
import React, { useState, useEffect } from "react";

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  return isOnline;
}
```

- Use in multiple components

```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return "Loading...";
  }

  return isOnline ? "Online" : "Offline";
}
```

```js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

- State of each component is completely independent, hooks are a way to reuse stateful logic and **not state itself**, so each call to a hook has a completely **isolated state** - so one can even use the same customHook twice in one component
- Custom hooks are more of a convention than a feature
  - If a function's name starts with "use" and it calls other hooks, it's said to be a custom hook
  - The `useSomething` naming convention is how linter plugin is able to find bugs in the code using hooks

## Other hooks

- E.g. `useContext` lets one subscribe to React context without introducing nesting

```js
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

- `useReducer` lets one manage local state of complex components with a reducer:

```js
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer);
  // ...
}
```