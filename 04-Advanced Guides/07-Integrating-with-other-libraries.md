# Integrating with other libraries

- React can be used in any web application
- It can be embedded in other applications and, with a little care, other applications can be embedded in React
- This guide will examine some of the more common use cases, focusing on integrating with `jQuery` and `Backbone`, but the same ideas can be applied to integrating other components with any existing code

## Integrating with DOM manipulations plugins

- React is unaware of changes made to the DOM outside of React
- It determines updates based on its own internal representation, and if the same DOM nodes are manipulated by another library, React gets confused and has no way to recover
- However, this doesn't mean it's impossible or even necessarily difficult to combine React with other ways of affecting the DOM, one just have to be mindful of what each is doing
- The easiest way to avoid conflicts is to prevent the React component from updating
- One can do this by rendering elements that React has no reason to update, like an empty `<div />`

### How to approach the problem

- Let's sketch out a wrapper for a generic jQuery plugin
- Will attach a ref to the root DOM element
- Inside `componentDidMount`, we'll get a reference to it so we can pass it to the jQuery plugin
- To prevent React from touching the DOM after mounting, we'll return an empty `<div />` from the `render()` method
- The `<div />` element has no properties or children, so React has no reason to update it, leaving the jQuery plugin free to manage that part of the DOM

```
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.somePlugin();
  }

  componentWillUmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}
```

- Many jQuery plugins attach event listeners to the DOM so it's important to detach them in `componentWillUnmount`
- If the plugin doesn't provide a method for cleanup, one will probably have to provide their own, remembering to remove any event listener to the plugin registered to prevent memory leaks

### Integrating with jQuery chosen plugin

- Let's write a minimal wrapper for the plugin `Chosen` which augments `<select>` inputs
- Note: Just because it's possible, it doesn't mean it's the best approach for React apps. It's still recommended to use React component when possible. React components are easier to reuse in React applications and often provide more control over their behavior and appearance
- Let's look at what Chosen does to the DOM
- If one call it on a `<select>` DOM node, it reads the attributes off of the original DOM node, hides it with an inline style, and then appends a separate DOM node with its own visual representation right after the `<select>`. Then it fires jQuery events to notify about the changes
- Let's say that this is the API we're striving for with our `<Chosen>` wrapper React component

```
function Example() {
  return (
    <Chosen onChange={value => console.log(value)}>
      <option>vanilla</option>
      <option>chocolate</option>
      <option>strawberry</option>
    </Chosen>
  );
}
```

- We'll implement it as an uncontrolled component for simplicity
- First, create an empty component with a `render()` method where it returns `<select>` wrapped in a `<div>`

```
class Chosen extends React.Component {
  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

- Notice how the `<select>` was wrapped in an extra `<div>`
- This is necessary because Chosen will append another DOM element right after the `<select>` node that's passed to it
- However, as far as React is concerned, `<div>` always only has a single child
- This is how we ensure that React updates won't conflict with the extra DOM node appended by Chosen
- It's important that if one modifies the DOM outside of React flow, one must ensure React doesn't have a reason to touch these DOM node
- Next, we'll implement the lifecycle methods 
- We need to initialize Chosen with the ref to the `<select>` node in `componentDidMount` and tear it down in `componentWillUnmount`

```
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();
}

componentWillUnmount() {
  this.$el.chosen('destroy');
}
```

- Note that React assigns no special meaning to the `this.el` field. It only works because we have previously assigned this field from a `ref` in the `render()` method

`<select className="Chosen-select" ref={el => this.el = el}>`

- This is enough to get our component to render, but we also want to be notified about the value changes
- We'll subscribe to the jQuery `change` event on the `<select>` managed by Chosen
- We won't pass `this.props.onChange` directly to Chosen because component's prop might change over time, and that includes event handlers
- Instead, we'll declare a `handleChange()` method that calls `this.props.onChange` and subscribe it to the jQuery `change` event

```
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();

  this.handleChange = this.handleChange.bind(this);
  this.$el.on('change', this.handleChange);
}

componentWillUnmount() {
  this.$el.off('change', this.handelChange);
  this.$el.chosen('destroy');
}

handleChange(e) {
  this.props.onChange(e.target.value);
}
```