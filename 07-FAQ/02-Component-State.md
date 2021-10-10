# Component State

- `setState` schedules an update to a component's `state` object
- When state changes, the components respond by re-rendering

## Difference between state and props

- `props` and `state` are both plain JS objects
- Both hold information and influences the output of render, but props get passed to components similar to function parameters whereas state is managed within the component similar to variables declared within a function

## Why is setState giving me the wrong value?

- In React, both `this.props` and `this.state` represent rendered values (i.e. what's on the screen)
- Calls to `setState` are async, so don't rely on `this.state` to reflect the new value immediately after calling `setState`
- Pass an updater function instead of an object if computing value based on current state

```js
incrementCount() {
  // This won't work as intended
  this.setState({ count: this.state.count + 1 });
}

handleSomething() {
  // Say this.state.count starts at 0
  this.incrementCount();
  this.incrementCount();
  this.incrementCount();

  // On re-render, this.state.count will be 1 instead of 3
  // This is due to the fact the incrementCount reads from this.state.count which ends up reading `this.state.count` as 0 every time
}
```

- Pass a function instead of an object to setState when depending on current state

## Difference between passing an object and a function in setState?

- Update function allows one to access the current state value inside the updater
- Since `setState` is batched, this lets one chain updates and ensure they build on top of each other

```js
incrementCount() {
  this.setState((state) => {
    return {count: state.count + 1}
  });
}
```

## When is setState async?

- `setState` is async inside event handlers
- If both Parent and Child call setState during a click event, Child isn't re-rendered twice
- Instead, React "flushes" the state updates at the end of the browser event

## Why doesn't React update this.state synchronously?

- As explained before, React intentionally "waits" until all components call `setState()` in their event handlers before starting to re-render. This boosts performance by avoiding unnecessary re-renders
- Why doesn't React just update `this.state` immediately without re-rendering?
  - This would break the consistency between `props` and `state`, causing hard to debug issues
  - New features become impossible to implement