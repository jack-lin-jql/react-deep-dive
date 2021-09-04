# Introducing hooks

- Hooks are a new addition in React 16.8 that lets one use state and other React features without a class

```
import React, { useState } from "react";

function Example() {
  // Declare a new state variable, which we'll call "count"
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

## No breaking changes

- 100% backwards-compatible, hooks don't contain any breaking changes
- Hooks provide a more direct API to the React concept one already know: props, state, context, refs, and lifecycle

## Motivation

- Class components make it hard to reuse stateful logic between components
  - React doesn't offer a way to "attach" reusable behavior to a component (e.g. connecting to a store)
  - Render props and HOC tries to solve this, but they require one to restruct components when using them
  - Hooks can extract stateful logic from a component so it can be tested independently and reused
  - Hooks allows one to reuse stateful logic without changing the component hierarchy

- Complex components become hard to understand
  - Each lifecycle method often contains a mix of unrelated logic
  - Mutually, related code that changes together gets split apart, but completely unrelated code ends up combined in a single method
  - Impossible to break these components into smaller ones since the stateful logic is all over the place
  - Difficult to test
  - One of the reasons why many people prefer to combine React with a separate state management library which may introduce too much abstraction
  - Hooks lets one split one component into smaller functions based on what pieces are related (such as setting up a subscription or fetching data)
  - Hooks doesn't force a split based on lifecycle methods

- Classes confuse both people and machines
  - Classes make code reuse and organization more difficult
  - Can be a large barrier to learning React
  - One has to understand how `this` works in JS, which is very different from how it works in most languages
  - One has to remember to bind event handlers
  - Classes don't minify well, makes hot reloading flaky and unreliable
  - Hooks allows one to use more of React's feature without classes
  