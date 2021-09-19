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

### Should I use one or many state variables? 

- Coming from class, one might be tempted to use a single state hook

```js
function Box() {
  const [state, setState] = useState({ left: 0, top: 0, width: 100, height: 100 });
  // ...
}
```

- When updating, manual merging is needed

```js
useEffect(() => {
  function handleWindowMouseMove(e) {
    setState(state => ({ ...state, left: e.pageX, top: e.pageY }));
  }

  window.addEventListener('mousemove', handleWindowMouseMove);
  return () => window.removeEventListener('mousemove', handleWindowMouseMove);
}, []);
```

- This is due to the fact when updating a state variable, it's different from `this.setState` in a class which merges updated fields into the object
- It's recommended to split state into multiple state variables based on which tend to change together
- E.g. split component state into `position` and `size` objects, this way we can always replace the `position` state without the need for merging

```js
function Box() {
  const [position, setPosition] = useState({ left: 0, top: 0 });
  const [size, setSize] = useState({ width: 100, height: 100 });

  useEffect(() => {
    function handleWindowMouseMove(e) {
      setPosition({ left: e.pageX, top: e.pageY });
    }
  })
}
```

- This also makes it easy to extract some related logic into custom hooks

```js
function Box() {
  const position = useWindowPosition();
  const [size, setSize] = useState({ width: 100, height: 100 });
}

function useWindowPosition() {
  const [position, setPosition] = useState({ left: 0, top: 0 });

  useEffect(() => {
    // ...
  }, []);

  return position;
}
```

- Both all state in a single useState call and having useState per each field can work, components tend to be most readable when one finds a balance between these two extremes, and group related state into a few independent state variables

### Run effect only on updates?

- one can use a mutable ref to manually store a boolean value corresponding to whether one is on the first or subsequent render then check that flag in the effect

### How to get the previous props or state?

- One can currently do it with a ref

```js
function Counter() {
  const [count, setCount] = useState(0);

  const prevCountRef = useRef();

  useEffect(() => {
    prevCountRef.current = count;
  });

  return <h1>Now: {count}, before: {prevCount}</h1>;
}
```

- One can extract it into a custom hook:

```js
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  return <h1>Now: {count}, before: {prevCount}</h1>;
}

function usePrevious(value) {
  const ref = useRef();

  useEffect(() => { ref.current.value; });

  return ref.current;
}
```

```js
// Works for props, state, or any other calculated value
function Counter() {
  const [count, setCount] = useState(0);

  const calculation = count + 100;
  count prevCalculation = usePrevious(calculation);
}
```

### Why am I seeing stale props or state inside my function?

- Any function inside a component, including event handlers and effects, "see" the props and state from the render it was created in

```js
function Example() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

- If one first click show alert and then increment the counter, the alert will show the `count` variable at the time one clicked the button. This prevents bugs caused by the code assuming props and state don't change
- If needed to read the latest state from some asynchronous callback, one could keep it in a ref, mutate it and read from it
- Another possible reason for stale props and states might be due to dependency array optimization but didn't correctly specify all the dependencies
  - E.g. if an effect specifies `[]` as the second argument but reads `someProp` inside, it will keep seeing the initial value of `someProp`
  - Either remove the dependency array or fix it

### How to implement getDerivedStateFromProps?

- You probably don't need it as one can update the state right during rendering
- React will re-run the component with updated state immediately after exiting the first render so it wouldn't be expensive

```js
function ScrollView({ row }) {
  const [isScrollingDown, setIsScrollingDown] = useState(false);
  const [prevRow, setPrevRow] = useState(null);

  if (row !== prevRow) {
    // Row changed since last render. Update isScrollingDown
    setIsScrollingDown(prevRow !== null && row > prevRow);
    setPrevRow(row);
  }

  return `Scrolling down: ${isScrollingDown}`;
}
```

### Is there forceUpdate?

- Both `useState` and `useReducer` hooks bail out of updates if the next value is the same as the previous one
- Mutating state in place and calling `setState` will not cause a re-render
- Normally, one shouldn't mutate local state in React, but as an escape hatch, one can use an incrementing counter to force a re-render even if the state hasn't changed

```js
const [ignored, forceUpdate] = useReducer(x => x + 1, 0);

function handlerClick() {
  forceUpdate();
}
```

### Can I make a ref to a function component?

- One may expose some imperative methods to a parent component with the `useImperativeHandle` hook, though, this shouldn't be needed often

### How can I measure DOM node?

- One rudimentary way to measure the position or size of a DOM node is to use a callback ref
- React will call that callback whenever the ref gets attached to a different node

```js
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measureRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```

- `useRef` wasn't used here since an object ref doesn't notify about changes to the current ref value
- Using a callback ref ensures that even if a child component displays the measured node later (e.g. in response to a click), it'll get notified about it in the parent component and can update the measurements
- Note the passing of `[]` to `useCallback`, this ensures the ref callback doesn't change between re-renders, so React won't call it unnecessarily
- In this example, the callback ref will be called only when the component mounts and unmounts since the rendered `h1` component stays present throughout any rerenders
- If one wants to be notified any time a component resizes, one may want to use ResizeObserver or a 3rd party hook built on it

- Let's extract to a custom hook

```js
function MeasureExample() {
  const [rect, ref] = useClientRect();
  return (
    <>
      <h1 ref={ref}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}

function useClientRect() {
  const [rect, setRect] = useState(null);
  const ref = useCallback(node => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, []);

  return [rect, ref];
}
```

