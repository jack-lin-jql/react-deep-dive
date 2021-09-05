# Using the statue hook

- Hooks let one use state and other React features without writing a class

```js
import React, { useState } from 'react';

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

## Equivalent class example

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({
          count: this.state.count + 1
        })}>Click me</button>
      </div>
    );
  }
}
```

## Hooks and function components

```js
// Function components
const Example = (props) => {
  // One can use Hooks here
  return <div />;
};
```

```js
// Also function components
function Example(props) {
  // One can use Hooks here
  return <div />;
}
```

- These might have previously been known as stateless components, but with Hooks, they are now simply function components
- Hooks don't work inside classes

## What's a hook?

```js
import React, { useState } from 'react';

function Example() {
  // ...
}
```

- What is a hook? - A special function that lets one hook into React features (e.g. React states)
- When would I use a hook? - When writing function components and in need of some states. Previously, one would have to convert to a class

## Declaring a state variable

```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
}
```

- In a function component, there's no `this`, so one can't assign or read `this.state` 

```js
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable
  const [count, setCount] = useState(0);
}
```

- What does calling useState do? - Declares a state variable
  - A way to preserve some value between the function calls - `useState` is a new way to use the exact same capabilities that `this.state` provides in a class
  - Normally, variables "disappear" after exiting but state variables are preserved by React
- What do we pass to useState as an argument? - Initial state
  - Unlike classes, the state doesn't have to be an object
  - Call useState() more than once for more than one state
- What does useState return? - Pair of values: the current state and a function that updates it
  - Similar to `this.state.count` and `this.setState`

- React will remember its current value between re-renders and provide the most recent one to the function
- If one wants to update the current `count`, we can call `setCount`

## Reading state

- When one wants to display the current count in a class

`<p>You clicked {this.state.count} times</p>`

- Use `count` directly in a function

`<p>You clicked {count} times</p>`

## Updating state

```js
// In class
<button onClick={() => this.setState({ count: this.state.count + 1 })}>
  Click me
</button>
```

```js
// In function
<button onClick={() => setCount(count + 1)}>
  Click me
</button>
```

## Recap

```js
1:  import React, { useState } from 'react';
 2:
 3:  function Example() {
 4:    const [count, setCount] = useState(0);
 5:
 6:    return (
 7:      <div>
 8:        <p>You clicked {count} times</p>
 9:        <button onClick={() => setCount(count + 1)}>
10:         Click me
11:        </button>
12:      </div>
13:    );
14:  }
```

- Line 1 imports the `useState` hook and allows a local state in a function
- Line 4 declares a new state variable
- Line 9 sets the count state with a new value, causing a re-render and passing the new `count` value to it

## Tip: Using multiple state variables

- One doesn't have to use many state variables since they can hold objects and arrays just fine to still allow grouping of related data together
- Unlike `this.setState` in a class, updating a variable always **replaces** the current state instead of merging it!
