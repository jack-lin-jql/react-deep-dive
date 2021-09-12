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