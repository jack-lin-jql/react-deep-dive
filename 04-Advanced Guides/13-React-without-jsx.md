# React without JSX

- JSX is not a requirement for using React
- Using React without JSX is especially convenient if one doesn't want to setup compilation in the build env
- Each JSX element is just syntactic sugar for calling `React.createElement(component, props, ...children)`. So, anything one can do with JSX can also be done with just plain JS

```
// JSX
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.toWhat}</div>;
  }
}

ReactDOM.render(
  <Hello toWhat="World" />,
  document.getElementById('root')
);
```

```
// Plain JS
class Hello extends React.Component {
  render() {
    return React.createElement('div', null, `Hello ${this.props.toWhat}`);
  }
}

ReactDOM.render(
  React.createElement(Hello, {toWhat: 'World'}, null),
  document.getElementById('root')
);
```

- See more examples of how JSX is converted to JS using the [online Babel compiler](https://babeljs.io/repl/#?presets=react&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA)
- The component can either be provided as a string, as a subclass of `React.Component`, or a plain function
- If one gets tired of typing `React.createElement` so much, a common pattern is to assign a shorthand:

```
const e = React.createElement;

ReactDOM.render(
  e('div', null, 'Hello World'),
  document.getElementById('root')
);
```

- If one uses this shorthand form of `React.CreateElement`, it can be almost as convenient as React without JSX
- Alternatively, one can refer to community projects such as `react-hyperscript` and `hyperscript-helpers` which offer a terser syntax