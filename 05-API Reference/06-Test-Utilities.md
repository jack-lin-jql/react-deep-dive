# Test utilities

- Importing

```
import ReactTestUtils from 'react-dom/test-utils'; // ES6
var ReactTestUtils = require('react-dom/test-utils'); // ES5 with npm
```

## Overview

- `ReactTestUtils` makes it easy to test React components in the testing framework of one's choice
- Note: It's recommended to use React Testing Library which is designed to enable and encourage writing tests that use components as the end users do
- For React v <= 16, the Enzyme library makes it easy to assert, manipulate, and traverse React Component's output
  - act()
  - mockComponent()
  - isElement()
  - isElementOfType()
  - isDOMComponent()
  - isCompositeComponent()
  - isCompositeComponentWithType()
  - findAllInRenderedTree()
  - scryRenderedDOMComponentsWithClass()
  - findRenderedDOMComponentsWithClass()
  - scryRenderedDOMComponentsWithTag()
  - findRenderedDOMComponentsWithTag()
  - scryRenderedDOMComponentsWithType()
  - findRenderedDOMComponentsWithType()
  - renderIntoDocument()
  - Simulate

## Reference

### act()

- To prepare a component for assertion, wrap the code rendering it and performing updates inside an `act()` call
- This makes the test run closer to how React works in the browser
- Note: If one uses react-test-renderer, it also provides an `act` export that behaves the same way

```
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { counter: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  handleClick() {
    this.setState(state => ({
      count: state.count + 1
    }));
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={this.handleClick}>
          Click me
        </button>
      </div>
    );
  }
}
```

```
import React from "react";
import ReactDOM from "react-dom";
import { act } from "react-dom/test-utils";
import Counter from "./Counter";

let container;

beforeEach(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
});

afterEach(() => {
  document.body.removeChild(container);
  container = null;
});

it ('can render and update a counter', () => {
  // Test first render and componentDidMount
  act(() => {
    ReactDOM.render(<Counter />, container);
  });

  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 0 times');
  expect(document.title).toBe('You clicked 0 times');

  // Test second render and componentDidUpdate
  act(() => {
    button.dispatchEvent(new MouseEvent('click', {bubbles: true}));
  });

  expect(label.textContent).toBe('You clicked 1 times');
  expect(document.title).toBe('You clicked 1 times');
});
```

- Don't forget that dispatching DOM events only work when the DOM container is added to the document. One can use a library like React Testing LIbrary to reduce the boilerplate code

### mockComponent()

```
mockComponent(
  componentClass,
  [mockTagName]
)
```

- Pass a mocked component module to this method to augment it with useful methods that allow it to be used as a dummy React component
- Instead of rendering as usual, the component will become a simple <div> (or other tag i mockTagName is provided) containing any provided children
- Note: mockComponent() is a legacy API, use jest.mock() instead

### isElement()

`isElement(element)`

- Returns `true` if `element` is any React element

### isElementOfType()

```
isElementOfType(
  element,
  componentClass
)
```

- Returns `true` if `element` is a React element whose type if of a React `componentClass`

### isDOMComponent()

`isDOMComponent(instance)`

- Returns true if instance is a DOM component such as a <div> or <span>

### isCompositeComponent()

`isCompositeComponent(instance)`

- Returns `true` if `instance` is a user-defined component, such as a class or a function

### isCompositeComponentWithType()

```
isCompositeComponentWithType(
  instance,
  componentClass
)
```

- Returns `true` if `instance` is a component whose type is of a React `componentClass`

