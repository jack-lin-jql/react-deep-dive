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