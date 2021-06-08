# State and lifecycle 

- From previous, one way to update the UI was through `ReactDOM.render()` to change the rendered output

```
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

- To make the `Clock` component truly reusable and encapsulated, it'll have to setup its own timer and update itself every second

```
// Start with encapsulating Clock
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock data={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```

- The above is missing the fact that `Clock` should set up a timer and updates the UI every second as its own implementation detail
- Ideally, the following should only be written once and have `Clock` update itself

```
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

- Here's where **state** comes to save the day
- State is similarly to props, but it's private and fully controlled by the component

## Converting a function to a class

- To convert a function component into a class, here's what one needs:
1. Create a ES6 class with the same name that extends React.Component
2. Add a single empty method to it called `render()`
3. Move the body of the function into the `render()` method
4. Replace `props` with `this.props` in the `render()` body
5. Delete the remaining empty function declaration

```
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}</h2>
      </div>
    );
  }
}
```

- The `render` method will be called each time an update happens, but as long as we render `<Clock />` into the same DOM node, only a single instance of the Clock class will be used, allowing us to use additional features such as local state and lifecycle methods

## Adding local state to a class

1. Replace `this.props.date` with `this.state.date` in the `render()` method

```
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  };
}
```

2. Add a class constructor to assign the initial `this.state`

```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

- Note the props passed in the constructor. Class components should always call the base constructor with `props`

3. Remove date prop from `<Clock />`

```
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

```
// Final result
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

## Adding lifecycle methods to a class

- It is very important to **free up resources taken by the components when they're destroyed** in applications with many components
- Mounting: Set up a timer whenever the Clock is rendered to the DOM for the first time (aka. mounting is when a component is rendered to the DOM)
- Unmounting: clear that timer whenever the DOM produced by the Clock is removed (aka. unmounting is when a component is removed from the DOM)
- One can declare special methods on the component class to run some code when a component mounts and unmounts

```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {}

  componentWillUnmount() {}
  
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

- These special methods are called "lifecycle methods"
- The `componentDidMount()` method **runs after the component output has been rendered to the DOM**

```
// A good place to setup a timer
componentDidMount() {
  this.timeID = setInterval(() => this.tick(), 1000);
}
```

- Though `this.props` is set up by React and `this.state` has a special meaning, one is free to add additional fields to the class manually when needing to store something private that doesn't need to participate in the data flow (since `this` is a JS context object at the end of the day)
- Now, to implement the `tick()` method which will run every second
- It'll use `this.setState()` to **schedule updates** to the component local state

```
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(() => this.tick(), 1000);
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

- The following is the order in which methods are called
1. When `<Clock />` is passed to `ReactDOM.render()`, React calls the constructor of `Clock`, initializing `this.state`
2. React then calls `Clock` component's `render()` method. This tells React what to display on the screen. React then updates the DOM to match the `Clock`'s render output
3. After the insertion of the `Clock` output into the DOM, React calls `componentDidMount()` lifecycle method. Within, the `Clock` component asks the browser to set up a timer calling `tick()` every second
4. Every time `tick()` is called, the `Clock` component **schedules** a UI update by calling `setState()` with an object containing the current time. Because of `setState()`, React knows the state has changed and calls the `render()` method again to learn what should be on the screen (what to update). In this case, `this.state.date` in the `render()` method will be different, so the render output will include the updated time. React updates the DOM accordingly
5. If the component is ever removed from the DOM, React calls `componentWillUnmount()` lifecycle method so the timer stops

## Using state correctly

- It breaks down to 3 things/rules

### Don't modify state directly

- The following will NOT re-render a component

```
// Wrong
this.state.comment = "Hello";
```

```
// Instead, use setState()
this.stateState({ comment: "Hello" });
```

- The only place where one can assign `this.state` is the constructor

### State updates may be asynchronous

- React maybe batch **multiple `setState()` calls into a single update** for performance
- Since `this.props` and `this.state` may be updated asynchronously, one should not rely on their value for calculating the next state

```
// E.g. this may fail to update the counter
this.setState({
  counter: this.state.counter + this.props.increment
});
```

- To fix, we can use the form of setState that accepts a function rather than an object. The function will receive the previous state as the first argument, and the props at the time the update is applied as the second arg

```
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));

// Without an arrow function
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### State updates are merged

- When calling `setState()`, React merges the object provided into the current state
- E.g. a state may contain several independent variables

```
constructor(props) {
  super(props);
  this.state = {
    posts: [],
    comments: []
  };
}
```

- One an then update them independently with separate `setState()` calls

```
componentDidMount() {
  fetchPosts().then(response => {
    this.setState({
      posts: response.posts
    });
  });

  fetchComments().then(response => {
    this.setState({
      comments: response.comments
    });
  });
}
```

- The **merging is shallow**, so `this.setState({ comments })` leaves `this.state.posts` intact, but completely replaces `this.state.comments`

## The data flows down

- Neither parent nor child components can know if a certain component is stateful or stateless, and they shouldn't care whether it's defined as a function or a class
- This is the reason why state is often called local or encapsulated. It's not accessible to any component other than the one that owns and sets it
- A component may choose to pass its state down as props to its child components

`<FormattedDate date={this.state.date} />`

- In the previous snippet, `FormattedDate` component will receive the `date` as a prop and wouldn't know whether it came from `Clock`'s state, prop, or typed by hand

```
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```

- This is commonly referred to as **top-down** or **unidirectional** data flow
- Any state is always owned by some specific component, and any data or UI derived from that state can only affect components *below* them in the tree
- Imagine a component tree as a waterfall of props, each component's state is like an additional water source that joins it at an arbitrary point but also flows down

```
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

- Each `Clock` here sets up its own timer and update independently
- In React apps,, stateful or stateless components are implementation details of the component that may change over time
- Stateless components can be used inside stateful components and vice versa