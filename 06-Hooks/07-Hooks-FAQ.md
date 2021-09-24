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

## Performance optimization

### Is it safe to omit function from the list of dependencies?

- Generally, no

```js
function Example({ someProp }) {
  function doSomething() {
    console.log(someProp);
  }

  useEffect(() => {
    doSomething();
  }, []); // This isn't safe since the function depends on `someProp`
}
```

- It's sometimes difficult to remember props and states that are used by functions outside of the effect which is why one will usually want to declare functions needed by an effect inside of it. Making it easy to see what values from the component scope the effect depends on

```js
function Example({ someProp }) {
  useEffect(() => {
    function doSomething() {
      console.log(someProp);
    }

    doSomething();
  }, [someProp]);
}
```

- If no values are used from the component scope, it's safe to specify `[]`

```js
useEffect(() => {
  function doSomething() {
    console.log('Hello');
  }

  doSomething();
}, []);
```

- When specifying a list of dependencies, it must include all values used inside the callback and participate in the React data flow, including props, state, and anything derived from them
- It's only safe to omit a function from the dependency list if nothing in it references props, state, or values derived from them

```js
// This example has a bug
function ProductPage({ productId }) {
  const [product, setProduct] = useState(null);

  async function fetchProduct() {
    const response = await fetch('..' + productId);
    const json = await response.json();
    setProduct(json);
  }

  useEffect(() => {
    fetchProduct();
  }, []); // Invalid since `fetchProduct` uses `productId`
}
```

- Recommended fix above is to move the function inside of the effect, making it easy to see which props and state the effect uses

```js
function ProductPage({ productId }) {
  const [product, setProduct] = useState(null);

  useEffect(() => {
    async function fetchProduct() {
      const response = await fetch('..' + productId);
      const json = await response.json();
      setProduct(json);
    }

    fetchProduct();
  }, [productId]);
}
```

- This also allows one to handle out-of-order responses with a local variable inside the effect

```js
useEffect(() => {
  let ignore = false;
  async function fetchProduct() {
    const response = await fetch('...' + productId);
    const json = await response.json();
    if (!ignore) setProduct(json);
  }

  fetchProduct();
  return () => { ignore = true };
}, [productId]);
```

- If one cannot move a function inside an effect, there're a few more options:
  - One can try moving the function outside of the component: In this case, the function is guaranteed to not reference any props or state, so it doesn't need to be in the list of dependencies
  - If the function is pure computation and safe to call while rendering, one may call it outside the effect instead and make the effect depend on the returned value
  - Last resort: Add a function to effect dependencies but wrap its definition into the useCallback hook. This ensures it doesn't change on every render unless its own dependencies also change

```js
function ProductPage({ productId }) {
  const fetchProduct = useCallback(() => {
    // ...
  }, [productId]);

  return <ProductDetails fetchProduct={fetchProduct} />;
}

function ProductDetail({ fetchProduct }) {
  useEffect(() => {
    fetchProduct();
  }, [fetchProduct]);
}
```

- Note above the need to keep the function in the dependency list since that ensures the change in productId triggers a refetch in the ProductDetails component

### What if the effect dependencies change too often?

- Omitting state from list of dependencies usually leads to bugs

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);

    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

- The problem here is that the `count` value goes stale due to the closure created on the value of `count` when it's `0`, so every second, the callback calls `setCount(0 + 1)`
- Specifying `[count]` would fix the bug but it would cause the interval to be reset on every re-render, so each `setInterval` would get one chance to execute before being cleared
- Fix this with the functional update form of setState

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(prevCount => prevCount + 1);
    }, 1000);

    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}
```

- The identity of `setCount` function is guaranteed to be stable so it's safe to omit
- Now, `setInterval` callback executes once a second and each time the inner call to `setCount` would have an up to date `count` value
- When more complex, one can also move the state update logic outside the effect with using useReducer. The identity of the dispatch function from useReducer is always stable even if the reducer function is declared inside the component and reads its props
- As last resort, one can have something like `this` in a class by using a ref to hold a mutable variable

```js
function Example(props) {
  const latestProps = useRef(props);
  useEffect(() => {
    latestProps.current = props;
  });

  useEffect(() => {
    function tick() {
      console.log(latestProps.current);
    }

    const id = setInterval(tick, 1000);
    return () => clearInterval(id);
  }, []);
}
```

### How to implement shouldComponentUpdate?

- One can wrap a function component with React.memo to shallowly compare its props

```js
const Button = React.memo((props) => {
  // Component
});
```

- This isn't a hook since it doesn't compose like hooks do
- `React.memo` is equivalent to `PureComponent`, but it only compares props
  - One can also add a second argument to specify a custom comparison function that takes the old and new props. Returning true skips the update
- `React.memo` doesn't compare states since there is no single state object to compare, but one can make children pure by optimizing with useMemo

### How to memoize calculations?

- The `useMemo` hook lets one cache calculations between multiple renders by "remembering" the previous computation 

```js
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

- Here, `computeExpensiveValue` is called and if the dependencies `[a, b]` haven't changed since the last value, `useMemo` skips calling it a second time and simply reuses the last value returned
- The function passes to `useMemo` runs during rendering, so don't do anything there one wouldn't normally do while rendering
- One may rely on useMemo as performance optimization, not as a semantic guarantee!
  - React may choose to forget some previous memoized values and recalculate on next render to free memory from offscreen components
  - Write code without `useMemo` and then add it to optimize performance
- `useMemo` also lets one skip an expensive re-render of a child

```js
function Parent({ a, b }) {
  // Only re-render if `a` changes
  const child1 = useMemo(() => <Child1 a={a} />, [a]);
  // Only re-render if `b` changes
  const child2 = useMemo(() => <Child2 b={b} />, [b]);

  return (
    <>
      {child1}
      {child2}
    </>
  );
}
```

- This structure won't work in a loop since Hooks can't be used there, but one can extract a separate component for the list item

### How to create expensive objects lazily?

-`useMemo` lets one memoize expensive calculation if the the dependencies are the same
- But that only serves as a hint, not a guarantee that the computation won't re-run
- Sometimes, one needs to be sure an object is only created once
- The first common use case is when creating the initial state is expensive:

```js
function Table(props) {
  // createRows() is called on every render
  const [rows, setRows] = useState(createRows(props.count));
  // ...
}
```

- To avoid re-creating the ignored initial state, one can pass a function to useState:

```js
function Table(props) {
  // createRow is only called once
  const [row, setRow] = useState(() => createRow(props.count));
  // ...
}
```

- React will only call this function during the first render
- One might occasionally want to avoid re-creating the useRef() initial value. E.g. ensure some imperative class instance only get created once

```js
function Image(props) {
  // IntersectionObserver is created on every render
  const ref = useRef(new IntersectionObserver(onIntersect));
  // ...
}
```

- `useRef` doesn't accept a special function overload like `useState`, instead one can write their own function that creates and sets lazily

```js
function Image(props) {
  const ref = useRef(null);

  function getObserver() {
    if(ref.current === null) {
      ref.current = new IntersectionObserver(onIntersect);
    }
    return ref.current;
  }

  // When needed it, call `getObserver()`
  // ...
}
```

### Are hooks slow due to the creation of functions in render?

- No. In modern browser, the raw performance of closures compared to classes doesn't differ significantly except in extreme scenarios
- Consider the design of hook is more efficient in a couple ways
  - Hooks avoid lots of overhead that classes require, like the cost of creating class instance and binding event handlers in the constructor
  - Idiomatic code using hooks doesn't need the deep component tree nesting that's prevalent in codebases that use HOCs, render props, and context. With smaller component tree, React has less work to do
- Traditionally, performance concerns around inline functions in React have been related to how passing new callbacks on each render breaks `shouldComponentUpdate` optimization in child components. Hooks approach this problem from 3 sides
  - The `useCallback` hook lets one keep the same callback reference between re-renders so that `shouldComponentUpdate` continues to work
  - The `useMemo` hook makes it easier to control when individual children update, reducing the need for pure components
  - `useReducer` hook reduces the need to pass callbacks deeply

### How to avoid passing callbacks down?

- Many do not enjoy manually passing callbacks through every level of a component tree, even though it's more explicit, it can feel like a lot of plumbing
- For larger components tree, one alternative is to pass down a dispatch function from `useReducer` via context

```js
const TodosDispatch = React.createContext(null);

function TodosApp() {
  // Note dispatch won't change between re-renders
  const [todos, dispatch] = useReducer(todosReducer);

  return (
    <TodosDispatch.Provider value={dispatch}>
      <DeepTree todos={todos} />
    </TodosDispatch.Provider>
  );
}
```

- Any child in the tree can use `dispatch` function to pass actions up to `TodosApp`

```js
function DeepChild(props) {
  // If to perform an action, single get dispatch from context
  const dispatch = useContext(TodosDispatch);

  function handleClick() {
    dispatch({ type: 'add', text: 'hello' });
  }

  return (
    <button onClick={handleClick}>Add todo</button>
  );
}
```

- This is both more convenient from the maintenance perspective (no need to keep forwarding callbacks) and avoids the callback problem altogether
- Passing `dispatch` down like this is recommended pattern for deep updates
- One can still choose to pass the application state down as props (more explicit) or as context (more convenient for very deep updates)
- If one uses context to pass down the state too, use two different context types - the `dispatch` context never changes, so components that read it don't need to rerender unless they also need the application state

### How to read an often-changing value from useCallback?

- Note: `dispatch` is recommended to be passed down by context rather than individual callback in props
- The following pattern may cause problems in concurrent mode, the safest solution right now is to invalidate the callback if some value it depends on changes
- If one need to memoize a callback with `useCallback` but it doesn't work very well due to the inner function being re-created too often
- If the function one is memoizing is an event handler and isn't used during rendering, one can use ref as an instance variable and save the last committed value into it manually

```js
function Form() {
  const [text, updateText] = useState('');
  const textRef = useRef();

  useEffect(() => {
    textRef.current = text; // Write it to the ref
  });

  const handleSubmit = useCallback(() => {
    const currentText = textRef.current; // Read it from the ref
    alert(currentText);
  }, [textRef]); // Don't recreate handleSubmit like [text] would do

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}
```

- One can also extract into a custom hook

```js
function Form() {
  const [text, updateText] = useState('');
  // Will be memoized even if `text` changes
  const handleSubmit = useEventCallback(() => {
    alert(text);
  }, [text]);

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}

function useEventCallback(fn, dependencies) {
  const ref = useRef(() => {
    throw new Error('Cannot call an event handler while rendering.');
  })

  useEffect(() => {
    ref.current = fn;
  }, [fn, ...dependencies]);

  return useCallback(() => {
    const fn = ref.current;
    return fn();
  }, [ref]);
}
```

- This isn't a recommended pattern!

## Under the hood

### How does React associate hook calls with components?

- React keeps track of the currently rendering component
- By the rules of hooks, it guarantees that hooks are only called from React components (or custom hooks) - which are also only called from React components)
- There is an internal list of "memory cells" associated with each component, which are just JS objects where one can put some data
- When a hook like useState gets called, it reads the current cell (or initializes it during the first render) and then moves the pointer to the next one
- That is how multiple `useState()` calls each get independent local state