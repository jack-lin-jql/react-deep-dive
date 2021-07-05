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

- Finally, there's one more thing to do. In React, props can change over time
- E.g., the `<Chosen>` component can get different children if parent component's state changes. This means that at integration points, it's important that we manually update the DOM in response to prop updates, since we no long let React manage the DOM for us
- Chosen's documentation suggests that we can use jQuery `trigger()` API to notify it about changes to the original DOM element
- We'll let React take care of updating `this.props.children` inside `<select>`, but we'll also add a `componentDidUpdate()` lifecycle method that notifies Chose about changes in the children list

```
componentDidUpdate(prevProps) {
  if(prevProps.children !== this.props.children) {
    this.$el.trigger("chosen:updated");
  }
}
```

- This way, Chosen will know to update its DOM element when the `<select>` children managed by React change

```
// Complete implementation
class Chosen extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.chosen();

    this.handleChange = this.handleChange.bind(this);
    this.$el.on('change', this.handleChange);
  }

  componentDidUpdate(prevProps) {
    if(prevProps.children !== this.props.children) {
      this.$el.trigger("chosen:updated");
    }
  }

  componentWillUnmount() {
    this.$el.off('change', this.handleChange);
    this.$el.chosen('destroy');
  }

  handleChange(e) {
    this.props.onChange(e.target.value);
  }

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

## Integrating with other view libraries

- React can be embedded into other applications thanks to the flexibility of ReactDOM.render()
- Although React is commonly used at startup to load a single root React component into the DOM, `ReactDOM.render()` can also be called multiple times for independent parts of the UI which can be as small as a button or as large as an app
- In fact, this is exactly how React is used at FB, it lets one write applications in React piece by piece, and combine them with their existing server-generated templates and other client-side code

### Replacing string-based rendering with React

- A common pattern in older web apps is to describe chunks of the DOM as a string and insert into the DOM like so `$el.html(htmlString)`
- These points in a codebase are perfect for introducing React, simply rewrite the string based rendering as a React component

```
// This implementation in jQuery
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```

```
// Can be re-written like this in React
function Button() {
  return <button id="btn">Say Hello</button>;
}

ReactDOM.render(
  <Button />,
  document.getElementById('container'),
  function() {
    $('#btn').click(function() {
      alert('Hello!');
    });
  }
);
```

- From here one can start moving more logic into the component and begin adopting more common React practices
- E.g., in component it's best not to rely on IDs because the same component can be rendered multiple times
- Instead, on can use the React event system and register the click handler directly on the React `<button>` element

```
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }

  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container');
);
```

- One can have as many such isolated components as needed and use `ReactDOM.render()` to render them to different DOM containers 


### Embedding React in a Backbone view

- Backbone views typically use HTML strings, or string-producing template functions, to create the content for their DOM elements
- This process, too, can be replaced with rendering a React component
- Let's create a Backbone view called `ParagraphView` 
  - It'll override Backbone's `render()` function to render a React `<Paragraph>` component into the DOM element provided by `this.el`

```
function Paragraph(props) {
  return <p>{props.text}</p>;
}

const ParagraphView = Backbone.View.extends({
  render() {
    const text = this.model.get('text');
    React.render(<Paragraph text={text} />, this.el);
    return this;
  },
  remove() {
    ReactDOM.unmountComponentAtNode(this.el);
    Backbone.View.prototype.remove.call(this);
  }
});
```

- It's important that we also call `ReactDOM.unmountComponentAtNode()` in the `remove` method so that React unregisters event handlers and other resources associated with the component tree when it's detached
- When a component is removed from within a React tree, the cleanup is performed automatically, but because we're removing the entire tree by hand, we must call this method