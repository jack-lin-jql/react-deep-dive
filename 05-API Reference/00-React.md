# React top-level API

- React is the entry point to the React library, if one loads React from a `<script>` tag, these top-level APIs are available on the `React` global
- If one uses ES6 with npm, one can write `import React from 'react"`
- If one uses ES5 with npm, one can write `var React = require('react')`

## Overview

### Components

- React components let one split the UI into independent, reusable pieces, and think about each piece in isolation
- React components can be defined by subclassing `React.Component` or `React.PureComponent`
  - React.Component
  - React.PureComponent
- If one doesn't use ES6 classes, one may use the `create-react-class` module instead
- React components can also be defined as functions which can be wrapped
  - React.memo

### Creating React elements

- It's recommended to use JSX to describe what one's UI should look like
- Each JSX element is just syntactic sugar for calling `React.createElement()`
- One will **not** typically invoke the following methods directly if using JSX
  - createElement()
  - createFactory()

### Transforming elements

- React provides several APIs for manipulating elements
  - cloneElement()
  - isValidElement()
  - React.Children

### Fragments

- React also provides a component for rendering multiple elements without a wrapper
  - React.Fragment

### REfs
  - React.createRef
  - React.forwardRef

### Suspense
- Suspense lets component "wait" for something before rendering
- Today, Suspense only supports one use case: loading components dynamically with React.lazy
- In the future, it'll support other use cases like data fetching
  - React.lazy
  - React.Suspense

### Hooks

- Hookes are a new addition in React 16.8
- They let one use state and other React features without writing a class
- Ref to its dedicated docs section

## Reference

### React.Component

- React.Component is the base class for React components where they are defined using ES6 classes

```
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

### React.PureComponent

- React.PureComponent is similar to React.Component
- The difference between that is React.Component doesn't implement `shouldComponentUpdate()`, but React.PureComponent implements it with a shallow prop and state comparison
- If one's React component's `render()` function renders the same result give the same props and state, one can use `React.PureComponent` for a performance boost in some cases
- Note: React.PureComponent's `shouldComponentUpdate()` only shallowly compare the objects. If these contain complex data structures, it may produce false-negatives for deeper differences. Only extend PureComponent when one expect to have simple props and state, or use `forceUpdate()` when one knows deep data structure have changed. Or consider using immutable objects to facilitate fast comparisons of nested data.
- Note: React.PureComponents' `shouldComponentUpdate()` skips prop updates for the whole component subtree. Make sure all the children components are also "pure"

### React.memo

```
const MyComponent = React.memo(function MyComponent(props) {
  /* render using props */
});
```

- React.memo is a HOC
- If one's component renders the same result given the same props, one can wrap it in a call to `React.memo` for a performance boost in some cases by memoizing the result
- This means that React will skip rendering the component, and reuse the last rendered result
- `React.memo` only checks for prop changes. If one's function component wrapped in `React.memo` has a useState, useReducer, or useContext hook in its implementation, it will still rerender when state or context change
- By default it'll only shallowly compare complex objects in the props object. If one wants control over the comparison, one can also provide a **custom comparison function as the second argument**

```
function MyComponent(props) {
  /* render using props */
}
function areEqual(prevProps, nextProps) {
  /*
    return true if passing nextProps to render would return the same result as passing prevProps to render, otherwise return false
  */
}

export default React.memo(MyComponent, areEqual);
```

- This method only exists as a performance optimization! **Do not rely on it to "prevent" a render as this can lead to bugs**
- Note: Unlike `shouldComponentUpdate()` method on class components, the `areEqual` function return true if the props are equal and false if the props are not equal. This is the inverse from `shouldComponentUpdate`

### createElement()

```
React.createElement(
  type,
  [props],
  [...children]
)
```

- Create and return a new React element of the given type
- The type argument can be either a tag name string (such as 'div' or 'span'), a React component type (a class or a function), or a React fragment type
- Code written with JSX will be converted to use `React.createElement()`
- One will not typically invoke `React.createElement()` directly if using JSX

### cloneElement()

```
React.cloneElement(
  element,
  [props],
  [...children]
)
```

- Clone and return a new React element using `element` as the starting point
- The resulting element will have the original element's props with the new props merged in shallowly
- New children will replace existing children
- `key` and `ref` from the original element will be preserved
- `React.cloneElement()` is almost equivalent to:

`<element.type {...element.props} {...props}>{children}</element.type>`

- However, it also preserves `ref`s. This means that if one gets a child with a `ref` on it, one won't accidentally steal it from your ancestor
- One will get the same `ref` attached to their new element

### createFactory()

`React.createFactory(type)`

- Returns a function that produces React elements of a given type
- Like `React.createElement()`, the type argument can be either a tag name string such as 'div' or 'span", a React component type (a class or function) or a React fragment type
- **This helper is considered legacy**, use JSX or use `React.createElement()` directly instead
- One will not typically use `React.createFactory()` directly if using JSX

### isValidElement()

`React.isValidElement(object)`

- Verifies the object is a React element, returns true or false

### React.Children


- `React.Children` provides utilities for dealing with the `this.props.children` opaque data structure

#### React.Children.map

`React.Children.map(children, function[(thisArg)])`

- Invokes a function on every immediate child contained within `children` with `this` set to `thisArg`
- If `children` is an array it will be traversed and the function will be called for each child in the array
- If children is `null` or `undefined`, this method will return `null` or `undefined` rather than an array
- Note: If `children` is a Fragment it will be treated as a single child and not traversed

#### React.Children.forEach

`React.Children.forEach(children, function[(thisArg)])`

- Like `React.Children.map()` but doesn't return an array

#### React.Children.count

`React.Children.count(children)`

- Returns the total number of components in `children`, equal to the number of times that a callback passed to `map` or `forEach` would be invoked

#### React.Children.only

`React.Children.only(children)`

- Verifies that children has only one child (a React element) and returns it. otherwise this method throws an error
- Note: `React.Children.only()` doesn't accept that return value of `React.Children.map()` because it's an array rather than a React element

#### React.Children.toArray

`React.Children.toArray(children)`

- Returns the `children` opaque data structure as a flat array with keys assigned to each child
- Useful if one wants to manipulate collections of children in their render methods, especially if one wants to reorder or slice `this.props.children` before passing it down
- Note: `React.Children.toArray()` changes keys to preserve the semantics of nested arrays when flattening lists of children. That is, `toArray` prefixes each key in the returned array so that each element's key is scoped to the input array containing it

### React.Fragment

- The `React.Fragment` component lets one return multiple elements in a `render()` method without creating an additional DOM element

```
render() {
  return (
    <React.Fragment>
      Some test.
      <h2>A heading</h2>
    </React.Fragment>
  );
}
```

- One can also use it with the shorthand `<></>` syntax

### React.createRef

- `React.createRef` creates a ref that can be attached to React elements via the ref attribute

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);

    this.inputRef = React.createRef();
  }

  render() {
    return <input type="text" ref={this.inputRef} />;
  }

  componentDidMount() {
    this.inputRef.current.focus();
  }
}
```

