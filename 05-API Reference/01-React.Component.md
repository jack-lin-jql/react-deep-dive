# React.Component

## Overview

- React lets one define components as classes or functions
- Components defined as classes currently provide more features
- To define a React component class, one needs to extend `React.Component`

```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

- The only method one must define in a `React.Component` subclass is the `render()` method, all other methods are optional
- It's strongly recommended against creating custom base component classes
- In React. code reuse is primarily achieved through composition instead of inheritance
- Note: React doesn't force one to use ES6 class syntax, if wanting to avoid, one can use `create-react-class` module or a similar custom abstraction instead

### The component lifecycle

- Each component has several "lifecycle methods" that one can override to run code at particular times in the process
- One can use [this diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) as a cheat sheet

#### Mounting

- These methods are called in the following order when an instance of a component is being created and inserted into the DOM
  - **constructor()**
  - static getDerivedStateFromProps()
  - **render()**
  - **componentDidMount()**
- Note: Legacy methods to avoid: UNSAFE_componentWillMount()

#### Updating

- An update can be caused by changes to props or state
- These methods are called in the following order when a component is being re-rendered:
  - static getDerivedStateFromProps()
  - shouldComponentUpdate()
  - **render()**
  - getSnapshotBeforeUpdate()
  - **componentDidUpdate()**
- Note: Legacy methods to avoid: UNSAFE_componentWillUpdate(), UNSAFE_componentWillReceiveProps()

#### Unmounting

- This method is called when a component is being removed from the DOM:
  - **componentWillUnmount()**

#### Error handling

- These methods are called when there's an error during rendering, in a lifecycle method or in the constructor of any child component
  - static getDerivedStateFromError()
  - componentDidCatch()

### Other APIs

- Each component also provides:
  - setState()
  - forceUpdate()

### Class properties

- defaultProps
- displayName

### Instance properties

- props
- state

## Reference

### Commonly used lifecycle methods

- The methods in this section covers the vast majority of use cases one will encounter creating React components

#### render()

- The render() method is the only required method in a class component
- When called, it should examine `this.props` and `this.state` and return one of the following types:
  - React element: Typically created via JSX. E.g. `<div />` and `<MyComponent />` are React elements that instruct React to render a DOM node, or another user-defined component, respectively
  - Arrays and fragments: Lets one return multiple elements from render
  - Portals: Lets one render children into a different DOM subtree
  - String and numbers: Rendered as text node in the DOM
  - Booleans or null: Renders nothing (exists to support `return test && <Child />` pattern where test is boolean)
- The `render()` function should be pure, meaning that it doesn't modify component state, it returns the same result each time it's invoked and it doesn't directly interact with the browser
- If one needs to interact with the browser, perform the work in `componentDidMount()` or the other lifecycle methods instead
- Keeping `render()` pure makes components easier to think about
- Note: `render()` won't be invoked if `shouldComponentUpdate()` returns false

### constructor()

`constructor(props)`

- If one doesn't initialize state and doesn't bind methods, one doesn't need to implement a constructor for their React component
- The constructor for a React component is called before it's mounted
- When implementing the constructor for a `React.Component` subclass, one should call `super(props)` before any other state. Otherwise, `this.props` will be undefined in the constructor which can lead to bugs
- Typically, React constructors are only used for two purposes:
  - Initializing `local state` by assigning an object to `this.state`
  - Binding event handler methods to an instance
- One should not call `setState()` in the `constructor()`, instead, if a component needs to use local state, assign the initial state to this.state directly in the constructor:

```
constructor(props) {
  super(props);
  // Don't call this.setState() here!
  this.state = { counter : 0 };
  this.handleClick = this.handleClick.bind(this);
}
```

- Constructor is the only place where one should assign `this.state` directly
- In all other methods, one needs to use `this.setState()` instead
- Avoid introducing any side-effects or subscriptions in the constructor, use `componentDidMount()` instead
- Note: Avoid copying props into state, this is a common mistake:

```
constructor(props) {
  super(props);
  // Don't do this!
  this.state = { color: props.color };
}
```

  - The problem is that it's both unnecessary (one can use `this.props.color` directly instead), and creates bugs (updates to the `color` prop won't be reflected in the state)
  - Only use this pattern if one intentionally wants to ignore prop updates
  - In that case, it makes sense to rename th prop to be called `initialColor` or `defaultColor`
  - One can then force a component to "reset" its internal state by changing its key when necessary

### componentDidMount()

- `componentDidMount()` is invoked immediately after a component is mounted (inserted into the tree)
- Initialization that requires DOM nodes should go here, if one needs to load data from a remote endpoint, this is a good place to instantiate the network request
- This method is a good place to set up any subscription. If one does that, don't forget to unsubscribe in `componentWillUnmount()`
- One may call `setState()` immediately in `componentDidMount()` which will trigger an extra rendering, but it'll happen before the browser updates the screen
- This guarantees that even though the `render()` will be called twice in this case, the user won't see the intermediate state
- Using this pattern with caution since it often causes performance issues
- In most cases, one should be able to assign the initial state in the `constructor()` instead
- It can, however, be necessary for cases like modals and tooltips when one needs to measure a DOM node before rendering something that depends on its size or position

### componentDidUpdate()

`componentDidUpdate(prevProps, prevState, snapshot)`

- `componentDidUpdate()` is invoked immediately after updating occurs
- **This method isn't called for the initial render**
- Use this as an opportunity to operate on the DOM when the component has been updated
- This is also a good place to do network requests as long as one compare the current props to previous props (e.g. a network request may not be necessary if the props haven't changed)

```
componentDidUpdate(prevProps) {
  // Typical usage (don't forget to compare props):
  if (this.props.userID !== prevProps.userID) {
    this.fetchData(this.props.userID);
  }
}
```

- One may call `setState()` immediately in `componentDidUpdate()` but note that it must be wrapped in a condition like the example above, or one will cause an infinite loop
- It would also cause an extra re-rendering which, while not visible to the user, can affect the component performance
- If one's trying to "mirror" some state to a prop coming from above, consider using the prop directly instead
- If the component implements the `getSnapshotBeforeUpdate()` lifecycle (which is rare), the value it returns will be passed as a third "snapshot" parameter to `componentDidUpdate()`. Otherwise this param will be undefined
- Note: `componentDidUpdate()` won't be invoked if `shouldComponentUpdate()` returns false

### componentWillUnmount()

- `componentWillUnmount()` is invoked immediately before a component is unmounted and destroyed
- Perform any necessary cleanup in this method, such as invalidating timers, canceling network requests, or cleaning up any subscription that were created in `componentDidMount()`
- One should NOT call `setState()` in `componentWillUnmount()` because the component will never be re-rendered
- Once a component instance is unmounted, it'll never be mounted again

### Rarely used lifecycle methods

- The methods in this section correspond to uncommon use cases, they're handy once in a while, but most of components probably don't need any of them

#### shouldComponentUpdate()

`shouldComponentUpdate(nextProps, nextState)`

- Use `shouldComponentUpdate()` to let React know if a component's output is not affected by the current change in state or props
- The default behavior is to re-render on every state change and in the vast majority of cases one should rely on the default behavior
- `shouldComponentUpdate()` is invoked before rendering when new props or state are being received
- Defaults to `true`. This method isn't called for the initial render or when `forceUpdate()` is used
- The method only exists as a performance optimization, do not rely on it to "prevent" a rendering, as this can lead to bugs
- Consider using the built-in **PureComponent** instead of writing `shouldComponentUpdate()` by hand
- `PureComponent` performs a shallow comparison of props and state, and reduces the chance that one will skip a necessary update
- If one is confident to write by hand, one can compare `this.props` with `nextProps` and `this.state` with `nextState` and return `false` to tell React the update can be skipped
- Note that returning `false` doesn't prevent child components from re-rendering when their state changes
- It's not recommended to do a deep equality check or using `JSON.stringify()` in `shouldComponentUpdate()` since it's very inefficient and will harm performance
- Currently, if `shouldComponentUpdate()` returns `false` then `UNSAFE_componentWillUpdate()`, `render()`, and `componentDidUpdate()` won't be invoked
- In the future, React may treat `shouldComponentUpdate()` as a hint rather than a strict directive and returning false may still result in a re-rendering of the component

#### static getDerivedStateFromProps()

`static getDerivedStateFromProps(props, state)`

- `getDerivedStateFromProps` is invoked right before calling the render method, both on the initial mount and on subsequent updates
- It should return an object to update the state, or `null` to update nothing
- This method exists for rare use cases where the state depends on changes in props over time
- E.g. it might be handy for implementing a `<Transition>` component that compares its previous and next children to decide which of them to animate in and out
- Deriving state leads to verbose code and makes components difficult to think about
  - If one needs to perform a side effect (e.g. data fetching or an animation) in response to a change in props, use `componentDidUpdate` lifecycle instead
  - If one wants to re-compute some data only when a prop changes, use a memoization helper instead
  - If one wants to "reset" some state when a prop changes, consider either making a component fully controlled or fully uncontrolled with a key instead
- This method doesn't have access to the component instance, if one would like, one can reuse some code between `getDerivedStateFromProps()` and the other class methods by extracting pure functions of the component props and state outside the class function
- Note this method is fired on every render, regardless of the cause. This is in contrast to `UNSAFE_componentWillReceiveProps`, which only fires when the parent causes a re-render and not as a result of the local `setState`

#### getSnapshotBeforeUpdate()

`getSnapshotBeforeUpdate(prevProps, prevState)`

- `getSnapshotBeforeUpdate()` is invoked right before the most recently rendered output is committed to e.g. the DOM
- It enables a component to capture some information from the DOM (e.g. scroll position) before it's potentially changed
- Any value returned by this lifecycle method will be passed as a parameter to `componentDidUpdate()`
- This use case isn't common, but it may occur in UIs like chat thread that need to handle scroll position in a special way
- A snapshot value (or null) should be returned

```
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the scroll position so it can be adjusted later
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }

    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    // (snapshot here if the value returned from getSnapshotBeforeUpdate)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```

- In the above example, it's important to read the `scrollHeight` property in `getSnapshotBeforeUpdate` because there may be delays between "render" phase lifecycles (like `render`) and "commit" phase lifecycles (like `getSnapshotBeforeUpdate` and `componentDidUpdate`)

#### Error boundaries

- Error boundaries are React components that catch JS errors anywhere in their child component tree, log those errors, and display a fallback UI instead of the component tree that crashed
- Error boundaries catch errors during rendering, in lifecycle methods, and in constructors of the whole tree below them
- A class component becomes an error boundary if it defines either (or both) of the lifecycle methods `static getDerivedStateFromError()` or `componentDidCatch()`
- Updating state from these lifecycles lets one capture an unhandled JS error in the below tree and display a fallback UI
- Only use error boundaries for recovering from unexpected exceptions; don't try to use them for control flow
- Note: Error boundaries only catch errors in the components **below** them in the tree. An error boundary can't catch an error within itself

##### static getDerivedStateFromError()

`static getDerivedStateFromError(error)`

- This lifecycle is invoked after an error has been thrown by a descendant component
- It receives the error that was thrown as a parameters and should return a value to update state

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      // One can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

- Note: `getDerivedStateFromError()` is called during the "render" phase, so side-effects aren't permitted. For those use cases, use `componentDidCatch()` instead

#### componentDidCatch()

`componentDidCatch(error, info)`

- This lifecycle is invoked after an error has been thrown by a descendant component
- It receives 2 params:
  1. `error` - The error that was thrown
  2. `info` - An object with a `componentStack` key containing information about which component threw the error
- `componentDidCatch()` is called during the "commit" phase, so side-effects are permitted. It should be used for things like logging errors:

```
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state os the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Example "componentStack":
    //    in ComponentThatThrows (created by App)
    //    in ErrorBoundary (created by App)
    //    in div (created by App)
    //    in App
    logComponentStackToMyService(info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // One can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

- Production and development builds of React slightly differ in the way `componentDidCatch()` handles errors
- On development, the errors will bubble up to `window`, this means that any `window.onerror` or `window.addEventListener('error', callback)` will intercept the errors that have been caught by `componentDidCatch()`
- On production, instead, the errors won't bubble up, which means any ancestor error handler will only receive errors not explicitly caught by `componentDidCatch()`
- Note: In the event of an error, one can render a fallback UI with `componentDidCatch()` by calling `setState`, but this will be deprecated in a future release. Use `static getDerivedStateFromError()` to handle fallback rendering instead

### Legacy lifecycle methods

- The lifecycle methods here still work, but are not recommended to use in code

#### UNSAFE_componentWillMount()

- NOTE: This was previously named `componentWillMount` and that name continues to work until version 17
- `UNSAFE_componentWillMount()` is invoked just before mounting occurs
- It's called before `render()`, therefore calling `setState()` synchronously in this method won't trigger an extra rendering
- Generally, it's commended to use the `constructor()` for state initialization
- Avoid introducing any side-effects or subscriptions in this method, use `componentDidMount()` instead
- THis is the only lifecycle method called on server rendering

#### UNSAFE_componentWillReceiveProps()

`UNSAFE_componentWillReceiveProps(nextProps)`

- NOTE: This lifecycle was previously named `componentWillReceiveProps` which will work until version 17
- NOTE: This lifecycle methods often leads to bugs and inconsistencies
  - If one needs to perform a side effect in response to changes in props, use `componentDidUpdate` lifecycle instead
  - If one used `componentWillReceiveProps` to re-compute some data only when a prop changes, use memoization helper instead
  - If one used `componentWillReceiveProps` to "reset" some state when a prop changes, consider either making a component full controlled or fully controlled with a key instead
- `UNSAFE_componentWillReceiveProps()` is invoked before a mounted component receives new props
- If one needs to update the state in response to prop changes, one may compare `this.props` and `nextProps` and perform state transitions using `this.setState()` in this method
- Note that if a parent component causes a component to re-render, this method will be called even if props have not changed. Make sure to compare the current and next values if one only wants to handle changes
- React doesn't call `UNSAFE_componentWillReceiveProps()` with initial props during mounting
- It only calls this method if some of component's prop may update
- Calling `this.setState()` generally doesn't trigger `UNSAFE_componentWillReceiveProps()`

#### UNSAFE_componentWillUpdate()

`UNSAFE_componentWillUpdate(nextProps, nextState)`

- Note: This lifecycle was previously named `componentWillUpdate` which will work until version 17
- `UNSAFE_componentWillUpdate()` is invoked just before rendering when new props or state are being received
- Use this as an opportunity to perform preparation before an update occurs
- This method is not called for the initial render
- Note that one cannot call `this.setState()` here, nor should one do anything else (e.g. dispatch a Redux action) that would trigger an update to a React component before `UNSAFE_componentWillUpdate()` returns
- Typically, this method can be replaced by `componentDidUpdate()`. If one was reading from the DOM in this method (e.g. to save a scroll position), one can move that logic to `getSnapshotBeforeUpdate()`
- Note: `UNSAFE_componentWillUpdate()` won't be invoked if `shouldComponentUpdate()` returns false

### Other APIs

- Unlike the lifecycle methods above, the methods below are methods that one can call from their components

#### setState()

`setState(updater, [callback])`

- `setState()` **enqueues** changes to the component state and tells React that this component and its children need to be re-rendered with the updated state
- This is the primary method one uses to update the UI in response to event handlers and server responses
- Think of `setState()` as **a request rather than an immediate command** to update the component
- For better perceived performance, React may delay it, and then update several components in a single pass
- React DOES NOT guarantee that the state changes are applied immediately
- `setState()` doesn't always immediately update the component. It may batch or defer the update until later
- This makes reading `this.state` right after calling `setState()` a potential pitfall
- Instead, one should use `componentDidUpdate` or a `setState` callback, either of which are guaranteed to fire after the update has been applied
- If one needs to set the state based on the previous state, refer to the updater argument
- `setState()` will always lead to a re-render unless `shouldComponentUpdate()` returns `false` 
- If mutable objects are being used and conditional rendering logic cannot be implemented in `shouldComponentUpdate()`, calling `setState()` only when the new state differs from the previous state will avoid unnecessary re-renders
- The first argument is an `updater` function with the signature:

`(state, props) => stateChange`

- `state` is a reference to the component state at the time the change is being applied
- It shouldn't be directly mutated, instead, changes should be represented by building a new object based on the input from `state` and `props`
- For instance, suppose we wanted to increment a value in state by `props.step`:

```
this.setState((state, props) => {
  return { counter: state.counter + props.step };
});
```

- Both `state` and `props` received by the updater function are guaranteed to be up-to-date
- The output of the updater is shallowly merged with `state`
- The second param to `setState()` is an optional callback function that will be executed once `setState` is completed and the component is re-rendered
  - Generally, it recommended to use `componentDidUpdate()` for such logic instead
- One may optionally pass an object as the first argument to `setState()` instead of a function:

`setState(stateChange[, callback])`

- This performs a shallow merge of `stateChange` into the new state, e.g., to adjust a shopping cart item quantity:

`this.setState({ quantity: 2 })`

- This form of `setState()` is also asynchronous, and multiple calls during the same cycle may be batched together
- E.g. if one attempts to increment an item quantity more than once in the same cycle, that will result in the equivalent of:

```
Object.assign(
  previousState,
  { quantity: state.quantity + 1 },
  { quantity: state.quantity + 1 },
  ...
)
```

- Subsequent calls will override values from previous calls in the same cycle, so the quantity will only be incremented once
- If the next state depends on the current state, it's recommended to use the updater function form instead:

```
this.setState((state) => {
  return {quantity: state.quantity + 1};
});
```

#### forceUpdate()

`component.forceUpdate(callback)`

- By default, when one component's state or prop changes, the component will re-render
- If a `render()` method depends on some other data, one can tell React that the component needs re-rendering by calling `forceUpdate()`
- Calling `forceUpdate()` will cause `render()` to be called on the component, skipping `shouldComponentUpdate()`
- This will trigger the normal lifecycle methods for child components, including the `shouldComponentUpdate()` method of each child
- React will still only update the DOM if the markup changes
- Normally, one should try to avoid all uses of `forceUpdate()` and only read from `this.props` and `this.state` in `render()`