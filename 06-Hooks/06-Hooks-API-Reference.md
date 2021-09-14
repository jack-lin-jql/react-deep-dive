# Hooks API reference

## Basic hooks

### useState

`const [state, setState] = useState(initialState);`

- Returns a stateful value and function to update it
- During initial render, the returned state is the same as the value passed as the first argument
- The `setState` function is used to update the state and it accepts a new state value to enqueue a re-render of the component

`setState(newState);`

- During subsequent re-renders, the first value returned by `useState` will always be the most recent state after applying updates
- React guarantees that `setState` function identity is stable and won't change on re-renders. This is why it's safe to omit from the `useEffect` or `useCallback` dependency list

### Functional updates

- If the new state is computed using the previous state, one can pass a function to `setState`
- The function will receive the previous value and return an updated value

```js
function Counter({ initialCount }) {
  const [count, setCount] = useState(initialCount);

  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

- Here the + and - use the functional form for updating since the updated value is based on the previous value
- If the update function returns the exact same value as the current state, the subsequent rerender will be skipped completely
- Note: Unlike the `setState` method found in class components, `useState` doesn't automatically merge update objects
  - One can replicate this behavior by combining the function updater form with object spread syntax
  
  ```js
    const [state, setState] = useState({});

    setState(prevState => {
      // Object.assign would also work
      return {...prevState, ...updatedValues};
    });
  ```

### Lazy initial state

- The `initialState` argument is the state used during the initial render
- In subsequent renders, it is disregarded
- If the initial state is the result of an expensive computation, one may provide a function instead, which will be executed only on the initial render:

```js
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);

  return initialState;
});
```

### Bailing out of a state update

- If one updates a state hook to the same value as the current state, React will bail out without rendering the children or firing effects (using the Object.is comparison algorithm)
- Note that React may still need to render that specific component again before bailing out
  - THis should not be a concern since React won't necessarily go deeper into the tree
  - If one is doing expensive calculations while rendering, one can optimize with useMemo

## useEffect

`useEffect(didUpdate);`

- Accepts a function that contains imperative possibly effectful code
- Mutations, subscriptions, timers, logging and other side effects are not allowed inside the main body of a function component (aka React's render phase) as doing so will lead to confusing bugs and inconsistencies in the UI
- `useEffect` takes a function and runs the effect after the render is committed to the screen, think of it as an escape hatch from React's purely functional world into the imperative world
- By default, effects run after every completed render, but one can choose to fire them only when certain values have changed

### Cleaning up an effect

- Effects often create resources that need to be cleaned up before the component leaves the screen, such as a subscription or timer ID
- The effect function may return a clean-up function

```js
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```

- The clean-up function runs before the component is removed from the UI to prevent memory leaks
- If a component renders multiple times (as they typically do), the previous effect is cleaned up before executing the next effect
- This means here a new subscription is created on every update

### Timing of effects

- Unlike `componentDidMount` or `componentDidUpdate`, the effect function fires after layout and paint, during a deferred event
- Making it suitable for many common side effects, like setting up subscription and event handlers, because most types of work shouldn't block the browser from updating the screen
- Not all effects can be deferred, e.g. DOM mutation visible to the user must fire synchronously before the next paint so that the user doesn't perceive a visual inconsistency (Similar to passive vs. active event listeners)
- For these types of effect, use `useLayoutEffect` which has the same signature as `useEffect`, but differs in when it's fired
- Though `useEffect` is deferred until after the browser has painted, it's guaranteed to fire before any new renders
- React will always flush a previous render's effect before starting a new update

### Conditionally firing an effect

- The default behavior for effects is to fire after every completed render
- That way an effect is always recreated if one of its dependencies changes
- However, it may be overkill to do so, e.g. new subscription on every update
- Pass a second argument to `useEffect` that's the array of values that the effect depends on to only run effects when the specified dependencies have updated

```js
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);
```

- The subscription will now only recreate when `props.source` changes
- Note: Make sure all values from the component scope (such as props and state) that change over time and that are used by the effect. Otherwise, stale values from previous renders will be referenced
- Pass `[]` when running an effect and clean up up only once
  - This tells React that the effect doesn't depend on any values from props or state, so it never needs to re-run
- When passing `[]`, props and state inside the effect will always have their initial values

- The array of dependencies isn't passed as arguments to the effect function. Conceptually, that's what they represent
  - Every value referenced inside the effect function should also appear in the dependency array

## useContext

`const value = useContext(MyContext);`

- Accepts a context object (value returned from `React.createContext`) and returns the current context value for that context
- The current context value is determined by the `value` prop of the nearest `<MyContext.Provider>` above the calling component in the tree
- When the nearest `<MyContext.Provider>` above the component updates, this hook will trigger a rerender with tha latest context `value` passed to that `MyContext` provider
  - Even if an ancestor uses `React.memo` or `shouldComponentUpdate`, a rerender will still happen starting at the component itself using `useContext`
- Don't forget the argument to `useContext` must be the context object itself:
  - Correct: useContext(MyContext)
  - Incorrect: useContext(MyContext.Consumer)
  - Incorrect: useContext(MyContext.Provider)
- A component calling `useContext` will always re-render when the context value changes
- Memoization can be used if re-rendering the component is expensive
- Tip: `useContext(MyContext)` only lets one read the context and subscribe to its changes. One still needs a `<MyContext.Provider>` above in the tree to provide the value for this context

### Putting it together with Context.Provider

```js
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

 const ThemeContext = React.createContext(themes.light);

 function App() {
   return (
     <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
     </ThemContext.Provider>
   );
 }

 function Toolbar(props) {
   return(
     <div>
      <ThemedButton>
     </div>
   );
 }

 function ThemedButton() {
   const theme = useContext(ThemeContext);

   return (
     <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
     </button>
   );
 }
```

## Additional hooks

- Following hooks are either variants of the basic ones from the previous section or only needed for specific edge cases

### useReducer

`const [state, dispatch] = useReducer(reducer, initialArg, init);`

- An alternative to `useState`
- Accepts a reducer of type `(state, action) => newState` and returns the current state paired with a `dispatch` method
- `useReducer` is usually preferable to `useState` when one have complex state logic that involves multiple sub-values or when the next state depends on the previous one
- `useReducer` also lets one optimize performance for components that trigger deep updates since one can pass dispatch down instead of callbacks

```js
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
    </>
  );
}
```

- Note: React guarantees that dispatch identify is stable and won't change on re-renders, which is why it's safe to omit from the `useEffect` or `useCallback` dependency list

#### Specifying initial state

- Two ways to initialize `useReducer` state
- Simplest way is to pass the initial state as a second argument

```js
const [state, dispatch] = useReducer(
  reducer,
  {count: initialCount}
);
```

- React doesn't use the `state = initialState` argument convention, so the initial value sometimes needs to depend on props and so is specified from the hook call instead

#### Lazy initialization

- One can also create the initial state lazily by passing an `init` function as the third argument
- The initial state will be set to `init(initialArg)`
- This lets one extract the logic for calculating the initial state outside the reducer
- Also handy for resetting the state later in response to an action

```js
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);

  return (
    <>
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>
        Reset
      </button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  )
}
```

#### Bailing out sf a dispatch

- If one returns the same value from a Reducer hook as the current state, React will bail out without rendering the children or firing effects
- Note react may still need to render that specific component again before bailing out
- This shouldn't be a concern since React won't necessarily go deeper into the tree

### useCallback

```js
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

- Returns a memoized callback
- Pass an inline callback and an array of dependencies
- Returns a memoized version of the callback that only changes if one of the dependencies has changed
- Useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. `shouldComponentDidUpdate`)
- `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`
- Note: The array of dependencies isn't passed as arguments to the callback

### useMemo

`const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);`

- Returns a memoized value
- Pass a "create" function and array array of dependencies
- `useMemo` will only recompute the memoized value when one of the dependencies has changed
- Helps avoid expensive calculations on every render
- Remember that function passed to `useMemo` runs during rendering, don't do anything that one wouldn't normally do while rendering. E.g. side effects that belong in `useEffect`
- If no array is provided, a new value will be computed on every render
- One may rely on useMemo as a performance optimization, not as a semantic guarantee
- React may choose to forget some previously memoized values and recalculate on next render

### useRef

`const refContainer = useRef(initialValue);`

- `useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument
- Returned object will persist across the full lifetime of the component

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };

  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

- `useRef` is like a "box" that can hold a mutable value in its `.current` property
- One might be familiar with refs primarily as a way to access the DOM
- If one pass a ref object to React with `<div ref={myRef} />`, **React will set its `.current` property to the corresponding DOM node whenever that node changes**
- However, `useRef()` is useful for more than the `ref` attribute
- It's also handy for keeping any mutable value around similar to how one would use instance field in classes
- This works since `useRef()` creates a plain JS object. The only difference between `useRef()` and creating a `{current: ...}` object by itself is that `useRef` will give one the same ref object on every render
- Keep in mind that `useRef` doesn't notify when its content changes, so mutating the `.current` property doesn't cause re-render
- If one wants to run some code when React attaches or detaches a ref to a DOM node, one may want to use callback ref instead

### useImperativeHandle

`useImperativeHandle(ref, createHandle, [deps])`

- `useImperativeHandle` customizes the instance value that is exposed to parent components when using `refs`
- This should be used with forwardRef

```js
function FancyInput(props, ref) {
  const inputRef = useRef();

  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />
}

FancyInput = forwardRef(FancyInput);
```

- Parent component that renders `<FancyInput ref={inputRef} />` would be able to call `inputRef.current.focus()`

### useLayoutEffect

- Identical in signature to `useEffect`, but fires synchronously after all DOM mutations
- Use this to read layout from the DOM and synchronously re-render
- Updates scheduled inside `useLayoutEffect` will be flushed synchronously, before the browser has a chance to paint
- Prefer the standard `useEffect` when possible to avoid blocking visual updates
- TIP: If one is migrating code from a class component, note `useLayoutEffect` fires in the same phase as `componentDidMount` and `componentDidUpdate`

### useDebugValue

`useDebugValue(value)`

- Used to display a label for custom hooks in React DevTools

```js
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // Show a label in DevTools next to this hook
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```

- Debug value is most valuable for custom hooks that are part of shared libraries

#### Defer formatting debug values

- Formatting a value for display might be an expensive operation and unnecessary unless a hook is actually inspected
- `useDebugValue` accepts a formatting function as an optional second param
- This function is only called if the hooks are inspected and receives the debug value as a parameter and should return a formatted display value
- E.g. a custom hook that returned a Date value could avoid calling the `toDateString` function unnecessarily by passing the following

`useDebugValue(date, date => date.toDateString());`