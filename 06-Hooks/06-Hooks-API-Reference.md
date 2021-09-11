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

