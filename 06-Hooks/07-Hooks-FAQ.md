# Hooks FAQ

## Adoption strategy

### Which versions of React include hooks?

- To enable hooks, all React packages need to be 16.8.0 or higher

### How much of React knowledge stays relevant

- Hooks are a more direct way to use the React features one already know (e.g. states, lifecycle, context and refs)

### Should I use hooks, classes or a mix of both?

- Hooks aren't allowed inside class components, one can definitely mix classes and function components with hooks in a single tree
- Whether a component is class or a function that uses hooks is an implementation detail of that component

### Do hooks cover all use cases for classes?

- There are currently no hook equivalent of `getSnapshotBeforeUpdate`, `getDerivedStateFromError`, and `componentDidCatch`

### Do hooks replace render props and HOCs?

- Render props and HOCs render only a single child. Hooks is a simpler way to serve this use case
- There are still places that may use `renderItem` prop, but in most cases, hooks will be sufficient and can help reduce nesting one's tree

### Redux connect and React Router?

- One can continue to use the same APIs

### Do hooks work with static typing?

- Hooks were designed with static typing in mind as they're functions, they're easier to type correctly than patterns like HOCs
- The latest Flow and TS React definitions include support for React hooks

### How to test components that use hooks?

- From React's PoV, a component using hooks is just a regular component
- If one's testing solution doesn't rely on React internals, testing components with hooks shouldn't be different from normally tested components

```js
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

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { act } 'react-dom/test-utils';
import Counter from "./Counter";

let container;

beforeEacht(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
});

afterEach(() => {
  document.body.removeChild(container);
  container = null;
});

it('can render and update a counter', () => {
  // test first render and effect
  act(() => {
    ReactDOM.render(<Counter />, container);
  });
  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 0 times');
  expect(document.title).toBe('You clicked 0 times');

  // Test second render and effect
  act(() => {
    button.dispatchEvent(new MouseEvent('click', {bubbles: true}));
  });

  expect(label.textContent).toBe('You clicked 1 times');
  expect(document.title).toBe('You clicked 1 times');
});
```

- The calls to `act` will also flush the effects inside of them
- If one needs to test a custom hook, one can do so by creating a component in the test and use the hook from it and test the component written

### Lint rules enforcement?

- Enforces rules of hooks by assuming that any function starting with `use` and a capital letter after it is a hook
- Enforces calls to hooks are either inside a `PascalCase` function (assumed to be a component) or another `useSomething` function (assumed to be a custom hook)
- Hooks are called in the same order on every render

## From classes to hooks

### How do lifecycle methods correspond to hooks?

- `constructor`: no need, one can initialize states with `useState`, if computation of the initial state is expensive, pass in a function to `useState`
- `getDerivedStateFromProps`: schedule an update while rendering instead
- `shouldComponentUpdate`: React.memo
- `render`: the function component body itself
- `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`: the `useEffect` hook can express all combinations of these
- `getSnapshotBeforeUpdate`, `componentDidCatch` and `getDerivedStateFromError`: there are no Hook equivalents for these methods yet, but they will be added soon.

### Is there something like instance variables?

- The `useRef()` hook isn't just for DOM refs
- The ref object is a generic container whose `current` property is mutable and can hold any value, similar to an instance property on a class

```js
function Timer() {
  const intervalRef = useRef();

  useEffect(() => {
    const id = setInterval(() => {...});
    intervalRef.current = id;

    return () => {
      clearInterval(intervalRef.current);
    }
  });
  // ...
}
```

- If just setting an interval, `ref` wouldn't be needed since the id can simply be local
- Useful if one wants to clear the interval from an event handler

```js
function handleCancelClick() {
  clearInterval(intervalRef.current);
}
```

- One can conceptually think of refs as similar to instance variables in a class
- Unless doing lazy initialization, avoid setting refs during rendering as it can lead to surprising behaviors, instead, modify refs in event handlers and effects