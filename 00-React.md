# React

## Philosophy

### Declarative

- Design simple views for each state in an application, React efficiently update and render just the right components when data changes
- Promotes predictability and ease of debugging

### Component-Based

- Create encapsulated components to manage their own states
- Component logic in JS allowing rich data passing while keeping state our of the DOM

## Intro

### A Simple Component

- React components implement a render() method which takes input data and returns what to display
- E.g. using CML-like syntax known as JSX. Input data passed into the component can be accessed by `render()` via `this.props`
- NOTE: JSX is **optional** and not required to use React

```
// Example JSX code
class HelloWorld extends React.Component {
  render() {
    return (
      <div>
        Hello {this.props.name}
      </div>
    )
  }
}

ReactDOM.render(
  <HelloWorld name="Jack" />,
  document.getElementById('hello-world');
);
```

```
// Previous JSX would get compiled into the following raw JS code
class HellWorld extends React.Component {
  render() {
    return React.createElement(
      "div",
      null,
      "Hello ",
      this.props.name
    );
  }
}

ReactDOM.render(React.createElement(HelloWorld, { name: "Jack" }), document.getElementById('hello-world'));
```

### A Stateful Component

- On top of taking input data through `this.props`, a component can maintain its own internal state via `this.state`
- When a component's state data changes, the rendered markup will be updated by re-invoking `render()`

```
class Timer extends React.Component {
  constructor(props) {
    super(props);
    this.state = { seconds: 0 };
  }

  tick() {
    this.setState(state => ({
      seconds: state.seconds + 1
    }));
  }

  componentDidMount() {
    this.interval = setInterval(() => this.tick(), 1000);
  }

  componentWillUnmount() {
    clearInterval(this.interval);
  }

  render() {
    return (
      <div>
        Seconds: {this.state.seconds}
      </div>
    );
  }
}

ReactDOM.render(
  <Timer />,
  document.getElementById('timer-example')
);
```

### An Application

- With `props` and `state`, one can put together a small Todo application
- The following example uses `state` to track the current list of items as well as the text that the user has entered

```
class TodoApp extends React.Component {
  constructor(props) {
    super(props);
    this.state = { items: [], text: '' };
    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  render() {
    return (
      <div>
        <h3>TODO</h3>
        <TodoList items={this.state.items} />
        <form onSubmit={this.handleSubmit}>
          <label htmlFor="new-todo">
            What needs to be done?
          </label>
          <input
            id="new-todo"
            onChange={this.handleChange}
            // This makes the input value controlled by the state.text
            value={this.state.text}
          />
          <button>
            Add #{this.state.items.length + 1}
          </button>
        </form>
      </div>
    );
  }

  handleChange(e) {
    // setState method updates passed key value pair instead of override the entire state with the object passed in
    this.setState({ text: e.target.value });
  }

  handleSubmit(e) {
    e.preventDefault();
    if (this.state.text.length === 0) {
      // Don't update state if empty entry
      return;
    }
    const newItem = {
      text: this.state.text,
      id: Date.now()
    };
    this.setState(state => ({
      items: state.items.concat(newItem),
      text: '' // reset the text in input field to blank
    }));
  }
}

class TodoList extends React.Component {
  render() {
    return (
      <ul>
        {this.props.items.map(item => (
          <li key={item.id}>{item.text}</li>
        ))}
      </ul>
    );
  }
}

ReactDOM.render(
  <TodoApp />,
  document.getElementById('todos-example')
);
```

### A Component Using External Plugins
- React allows one to interface with other libraries and frameworks
- Remarkable here is an external Markdown library

```
class MarkdownEditor extends React.Component {
  constructor(props) {
    super(props);
    this.md = new Remarkable();
    this.handleChange = this.handleChange.bind(this);
    this.state = { value: 'Hello, **world**!' };
  }

  handleChange(e) {
    this.setState({ value: e.target.value });
  }

  getRawMarkup() {
    return { __html: this.md.render(this.state.value) };
  }

  render() {
    return (
      <div className="MarkdownEditor">
        <h3>Input</h3>
        <label htmlFor="markdown-content">
          Enter some markdown
        </label>
        <textarea
          id="markdown-content"
          onChange={this.handleChange}
          defaultValue={this.state.value}
        />
        <h3>Output</h3>
        <div
          className="content"
          dangerouslySetInnerHTML={this.getRawMarkup()}
        />
      </div>
    );
  }
}

ReactDOM.render(
  <MarkdownEditor />,
  document.getElementById('markdown-example')
);
```