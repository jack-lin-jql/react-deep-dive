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