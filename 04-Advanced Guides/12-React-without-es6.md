# React without ES6

- Normally one would define a React component as a plain JS class:

```
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

- If one doesn't use ES6 yet, one may use the `create-react-class` module instead:

```
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
```

- The API of ES6 classes is similar to `createReactClass()` with a few exceptions

## Declaring default props

- With functions and ES6 classes `defaultProps` is defined as a property on the component itself:

```
class Greeting extends React.Component {
  //...
}

Greeting.defaultProps = {
  name: 'Mary'
};
```

- With `createReactClass()`, one needs to define `getDefaultProps()` as a function on the passed object:

```
var Greeting = createReactClass({
  getDefaultProps: function() {
    return {
      name: 'Mary'
    };
  },
  // ...
});
```

## Setting the initial state

- In ES6 classes, one can define the initial state by assigning `this.state` in the constructor:

```
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: props.initialCount};
  }
  // ...
}
```

- With `createReactClass()`, one have to provide a separate `getInitialState` method that returns the initial state

```
var Counter = createReactCLass({
  getInitialState: function() {
    return {count: this.props.initialCount}
  },
  //...
});
```

## Autobinding

- In React components declared as ES6 classes, methods follow the same semantics as regular ES6 classes
- This means that they don't automatically bind `this` to the instance
- You'll have to explicitly use `.bind(this)` in the constructor:

```
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
    // This line is important!
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    alert(this.state.message);
  }

  render() {
    // Because `this.handleClick` is bound, we can use it as an event handler
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

- With `createReactClass()`, this isn't necessary since it binds all methods:

```
var SayHello = createReactClass({
  getInitialState: function() {
    return {message: 'Hello!'};
  },

  handleClick: function() {
    alert(this.state.message);
  },

  render: function {
    return (
      <button onClick={this.handleClick}>
        Say Hello
      </button>
    );
  }
});
```

- This means writing ES6 classes comes with a little more boilerplate code for event handlers, but the upside is slightly better performance in large applications
- If the boilerplate code is too unattractive, one may enable the experimental Classes Properties syntax proposal with Babel:

```
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
  }

  // WARNING: this syntax is experimental!
  // Using an arrow here binds the method:
  handleClick = () => {
    alert(this.state.message);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

- Please note that the syntax above is experimental and the syntax may change, or the proposal might not make it into the language
- To play safe, here are a few other options:
  - Bind methods in constructor
  - Use arrow functions, e.g. `onClick={(e) => this.handleClick(e)}
  - Keep using `createReactClass`

## Mixins

- Note: ES6 launched without any mixin support, so there is no support for mixins when using React with ES6 classes
- Codebases using mixins usually have numerous issues, so using this isn't recommended
- Refer to official documentation for this section.