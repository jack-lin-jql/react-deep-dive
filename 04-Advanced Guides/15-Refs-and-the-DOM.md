# Refs and the DOM

- Refs provide a way to access DOM nodes or React elements created in the render method
- In the typical React dataflow, `props` are the only way the parent components interact with their children
  - To modify a child, one re-render it with new props
- However, there are a few cases where on need to imperatively modify a child outside of the typical dataflow
- The child to be modified could be an instance of a React component, or it could be a DOM element. For both of these cases, React provides an escape hatch

## When to use refs

- A few good use cases:
  - Managing focus, text selection, or media playback
  - Triggering imperative animations
  - Integrating with third-party DOM libraries
- Avoid using refs for anything that can be done declaratively
- E.g. instead of exposing `open()` and `close()` methods on a `Dialog` component, pass an `isOpen` prop to it

## Don't overuse refs

- One's first inclination may be to use refs to "make things happen" in their app
- If this is the case, take a moment and think more critically about where state should be owned in the component hierarchy
- Often, it becomes clear that the proper place to "own" that state is at a higher level in the hierarchy, so simply lift the state up
- Note: It's recommended for earlier versions (before 16.3) of React to use callback refs instead of `React.createRef()`

## Creating refs

- Refs are created using `React.createRef()` and attached to React elements via the `ref` attribute
- Refs are commonly assigned to an instance property when a component is constructed so they can be referenced throughout the component

```
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }

  render() {
    return <div ref={this.myRef} />;
  }
}
```

## Accessing refs

- When a ref is passed to an element in `render`, a reference to the node becomes accessible at the `current` attribute of the ref

`const node = this.myRef.current;`

- The value of the ref differs depending on the type of the node:
  - When the `ref` attribute is used on an HTML element, the `ref` created in the constructor with `React.createRef()` receives the underlying DOM element as its `current` property
  - When the `ref` attribute is used on a custom class component, the `ref` object receives **the mounted instance of the component as its `current`**
  - One may **not** use the ref attribute on function components because they don't have instances

### Adding a ref to a DOM element

- The following uses a `ref` to store a reference to a DOM node:

```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // create a ref to store the textInput DOM element
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // Explicitly focus the text input using the raw DOM API
    // Note: we're accessing "current" to get the DOM node
    this.textInput.current.focus();
  }

  render() {
    // tell React that we want to associate the <input> ref with the `textInput` that we created in the constructor
    return (
      <div>
        <input
          type="text"
          ref={this.textInput}
        />
        <input
          type="button"
          value="Focus the next input"
          onClick={this.focusTextInput}
        />
      </div>
    )
  }
}
```

- React will assing the `current` property with the DOM element when the component counts, and assign it back to `null` when it unmounts
- `ref` updates happen before `componentDidMount` or `componentDidUpdate` lifecycle methods

### Adding a ref to a class component

- If one wanted to wrap the `CustomTextInput` above to simulate being clicked immediately after mounting, we could use a ref to get access to the custom input and call its `focusTextInput` method manually

```
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  componentDidMount() {
    // Note here that this is literally the reference to the instance of the component CustomTextInput, NOT the actual DOM
    this.textInput.current.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
```

- Note that this only works if `CustomTextInput` is declared as a class:

```
class CustomTextInput extends React.Component {
  // ...
}
```

### Refs and function components

- By default, one may not use the ref attribute on function components since they don't have instances:

```
function MyFunctionComponent() {
  return <input />;
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  render() {
    // this will NOT work
    return <MyFunctionComponent ref={this.textInput} />;
  }
}
```

- If one wants to allow people to take a `ref` to their function component, one can use `forwardRef` (possibly in conjunction with `useImperativeHandle`) or one can convert the component to a class
- One can, however, use the ref attribute inside a function component as long as one refer to a DOM element or a class component:

```
function CustomTextInput(props) {
  // textInput must be declared here so the ref can refer to it
  const textInput = useRef(null);

  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput}
      />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```

## Exposing DOM refs to parent components

- In rare cases, one might want to have access to a child's DOM node from a parent component
- This is generally not recommended because it breaks component encapsulation, but it can occasionally be useful for triggering focus or measuring the size of position of a child DOM node
- While one could add a ref to the child component, this is not an ideal solution, as one would only get a component instance rather than a DOM node. Additionally, this wouldn't work with function components
- If using React 16.3 or higher, it's recommended to use ref forwarding for these cases. Ref forwarding lets components opt into exposing any child component's ref as their own
- See forwarding refs for detailed documentation
- If React 16.2 or lower, or if one needs more flexibility than provided by ref forwarding, one can explicitly pass a ref as a differently named prop
- When possible, it's advised against exposing DOM nodes, but it can be a useful escape hatch. Note that this approach requires one to add some code to the child component. If one has no control over the child component implementation, the last option is to use `findDOMNode()`, but it's discouraged and deprecated in StrictMode

## Callback refs

- React also supports another way to set refs called "callback refs", which gives more fine-grain control over when refs are set and unset
- Instead of passing a `ref` attribute created by `createRef()`, one pass a function
- The function receives the React component instance or HTML DOM element as its argument, which can be stored and accessed elsewhere
- The example below implements a common pattern: using the `ref` callback to store a reference to a DOM node in an instance property

```
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // Focus the text input using the raw DOM API
      if (this.textInput) this.textInput.focus();
    }
  }

  componentDidMount() {
    // autoFocus the input on mount
    this.focusTextInput();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM element in an instance field (e.g. this.textInput)
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

- React will call the `ref` callback with the DOM element when the component mounts, and call it with `null` when it unmounts
- Refs are guaranteed to be up-to-date before `componentDidMount` or `componentDidUpdate` fires
- One can pass callback refs between components like one can with object refs that were created with `React.createRef()`

```
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
```

- In the example above, `Parent` passes its ref callback as an `inputRef` prop to the `CustomTextInput`, and the `CustomTextInput` passes the same function as a special `ref` attribute to the `<input>`
- As a result, `this.inputElement` in `Parent` will be set to the DOM node corresponding to the `<input>` element in the `CustomTextInput`

## Caveats with callback refs

- If the `ref` callback is defined as an inline function, it'll get called twice during updates, first with `null` and then again with the DOM element
- This is because a new instance of the function is created with each render, so React needs to clear the old ref and set up a new one
- One can avoid this by defining the `ref` callback as a bound method on the class, but note that it shouldn't matter in most cases