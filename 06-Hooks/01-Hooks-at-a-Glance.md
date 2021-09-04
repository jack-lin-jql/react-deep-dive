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
- 