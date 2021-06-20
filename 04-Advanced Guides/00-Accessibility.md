# Accessibility

## Why accessibility?

- Web accessibility also referred to a11y is the design and creation of website that can be used by every one
- Accessibility support is necessary to allow assistive technology to interpret web pages
- React fully supports building accessible websites, often by using standard HTML techniques

## Standards and guidelines 

### WCAG - Web Content Accessibility Guidelines 

- WCAG provides guidelines for creating accessible web sites

### WAI-ARIA - Web Accessibility Initiative - Accessible Rich Internet Application

- The WAI-ARIA document contains techniques for building fully accessible JS widgets
- Note that all `aria-*` HTML attributes are fully supported in JSX. Whereas most DOM properties and attributes in React are camelCased, these attributes should by hyphen-cased as they are in plain text

```
<input
  type="text"
  aria-label={labelText}
  aria-requires="true"
  onChange={onChangeHandler}
  value={inputValue}
  name="name"
/>
```

## Semantic HTML

- Semantic HTML is the foundation of accessibility in a web app
- Using various HTML elements to reinforce the meaning of information in our websites will often give us accessibility for free
- At times, HTML semantics are broken when adding `<div>` elements to JSX to make React code work, especially when working with lists (`<ol>`, `<ul>`, and `<dl>`) and the HTML `<table>`
- In these cases, one should rather use React Fragments to group together multiple elements

```
import React, { Fragment } from 'react';

function ListItem({ item }) {
  return (
    <Fragment>
      <dt>{item.term}</dt>
      <dt>{item.description}</dt>
    </Fragment>
  );
}

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        <ListItem item={item} key={item.id} />
      ))}
    </dl>
  );
}
```

- One can map a collection of items to an array of fragments as one would any other type of elements as well

```
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Fragments should also have a `key` prop when mapping collections
        <Fragment key={item.id}>
          <dt>{item.term</dt>
          <dt>{item.description</dt>
        </Fragment>
      ))}
    </dl>
  );
}
```

- When one doesn't need any props on the Fragment tag, one can use the short syntax if supported by tooling

```
function ListItem({ item }) {
  return (
    <>
      <dt>{item.term}</dt>
      <dd>{item.description</dd>
    </>
  );
}
```

## Accessible forms 

### Labels 

- Every HTML form control such as `<input>` and `<textarea>` need to be labeled accessibly
- One needs to provide descriptive labels that are also exposed to screen readers
- Though the standard HTML practices can be directly used in React, note that `for` attribute is written as `htmlFor` in JSX

```
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name" />
```

- Resources [here](https://reactjs.org/docs/accessibility.html#labeling)

### Notifying the user of errors

- Error situations need to be understood by all users
- Resources [here](https://reactjs.org/docs/accessibility.html#notifying-the-user-of-errors)

## Focus Control 

- Ensure a web app can be fully operated with the keyboard only
- Resources [here](https://reactjs.org/docs/accessibility.html#focus-control)

### Keyboard focus and focus outline

- Keyboard focus refers to the current element in the DOM that is selected to accept input from the keyboard
- Only use CSS to remove this outline (`outline: 0`) if replacing it with another focus outline implementation

### Mechanisms to skip to desired content

- Provide a mechanism to allow users to skip past navigation sections in an application as this assists and speeds up keyboard navigation
- Skiplinks or Skip Navigation Links are hidden navigation links that only become visible when keyboard user interact with the page. They're easy to implement with internal page anchors and some styling
- Resource [here](https://webaim.org/techniques/skipnav/)
- Also use landmark elements and roles such as `<main>` and `<aside>` to demarcate page regions as assistive technology allow the user to quickly navigate these sections
- Read more [here](https://www.scottohara.me/blog/2018/03/03/landmarks.html)

### Programmatically managing focus

- React applications continuously modify the HTML DOM during runtime, sometimes leading to keyboard focus being lost or set to an unexpected element
- To repair this, one needs to programmatically nudge the keyboard focus in the right direction
- E.g. by resetting keyboard focus to a button that opened a modal window after that modal window is closed
- Learn from MDN about building keyboard navigable JS widgets [here](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Keyboard-navigable_JavaScript_widgets)
- One can use Refs to DOM elements to set focus in React

```
// Define here
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // Create a ref to store the textInput DOM element
    this.textInput = React.createRef();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM element in an instance field (for example, this.textInput)

    return (
      <input
        type="text"
        ref={this.textInput}
      />
    );
  }
}

// To be used elsewhere as follows
focus() {
  // Explicitly focus the text input using the raw DOM API
  // Note: accessing "current" to get the DOM node
  this.textInput.current.focus();
}
```

- Sometimes, a parent component needs to set focus on an element in a child component. One can do this by exposing DOM refers to parent components through a special prop on the child component that forwards the parent's ref to the child's DOM node

```
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.inputElement = React.createRef();
  }

  render() {
    return (
      <CustomTextInput inputRef={this.inputElement} />
    );
  }
}

// Now onw can set focus when required
this.inputElement.current.focus();
```

- When using HOC to extend components, it's recommended to forward the ref to the wrapped component using `forwardRef` function of React
- If a third party HOC doesn't implement ref forwarding, the above pattern can still be used as a fallback
- Great example focus management example [here](https://github.com/davidtheclark/react-aria-modal)

## Mouse and pointer events

- Ensure that all functionality exposed through a mouse or pointer event can also be accessed using the keyboard alone
- Depending only on pointer device will lead to many cases where keyboard users cannot use an application
- A prolific example of broken accessibility caused by click events is the outside click pattern, where a user can disable an opened popover by clicking outside the element
- Typically implemented by attaching `click` event to the `window` object that closes the popover

```
class OuterClickExample extends React.Component {
  constructor(props) {
    super(props);
    
    this.state = { isOpen: false };
    this.toggleContainer = React.createRef();

    this.onClickHandler = this.onClickHandler.bind(this);
    this.onClickOutsideHandler = this.onClickOutsideHandler(this);
  }

  componentDidMount() {
    window.addEventListener('click', this.onClickOutsideHandler);
  }

  componentWillUnmount() {
    window.removeEventListener('click', this.onClickOutsideHandler);
  }

  onClickHandler() {
    this.setState(currentState => ({
      isOpen: !currentState.isOpen
    }));
  }

  onClickOutsideHandler(event) {
    if (this.state.isOpen && !this.toggleContainer.current.container(event.target)) {
      this.setState({ isOpen: false });
    }
  }

  render() {
    return (
      <div ref={this.toggleContainer}>
        <button onClick={this.onClickHandler}>Select an option</button>
        {this.state.isOpen && (
          <ul>
            <li>Option 1</li>
            <li>Option 2</li>
            <li>Option 3</li>
          </ul>
        )}
      </div>
    );
  }
}
```

- This may work fine for users with pointer devices, but operating this with keyboard alone leads to broken functionality when tabbing to the next element as the `window` object never receives a `click` event, leading to obscured functionality which blocks users from using the app
- The same functionality can be achieved by using appropriate event handlers instead such as `onBlur` and `onFocus`

```
class BlurExample extends React.Component {
  constructor(props) {
    super(props);

    this.state = { isOpen: false };
    this.timeOutId = null;

    this.onClickHandler = this.onCLickHandler.bind(this);
    this.onBlurHandler = this.onBlurHandler.bind(this);
    this.onFocusHandler = this.onFocusHandler.bind(this);
  }

  onClickHandler() {
    this.setState(currentState => ({
      isOpen: !currentState.isOpen
    }));
  }

  // We close the popover on the next tick by using setTimeout. This is necessary because we need to first check if another child of the element has received focus as the blur event fires prior to the new focus event
  onBlurHandler() {
    this.timeOutId = setTimeout(() => {
      this.setState({
        isOpen: false
      });
    });
  }

  // If a child receives focus, do not close the popover
  onFocusHandler() {
    clearTimeout(this.timeoutId);
  }

  render() {
    // React assists by bubbling the blur and focus event to the parent
    return (
      <div
        onBlur={this.onBlueHandler}
        onFocus={this.onFocusHandler}
      >
        <button
          onClick={this.onClickHandler}
          aria-haspopup="true"
          aria-expanded={this.state.isOpen}
        >
          Select an option
        </button>
        {this.state.isOpen && (
          <ul>
            <li>Option 1</li>
            <li>Option 2</li>
            <li>Option 3</li>
          </ul>
        )}
      </div>
    );
  }
}
```

- This code exposes the functionality to both pointer device and keyboard users
- Note the added `aria-*` props to support screen-read users
- The keyboard events to enable arrow key interaction of the popover option have not been implemented for simplicity's sake


## More complex widgets

- A more complex UX should not mean less accessible one
- Whereas accessibility is most easily achieved by coding as close to HTML as possible, even the most complex widgets can be coded accessibly
- [ARIA roles](https://www.w3.org/TR/wai-aria/#roles) and [ARIA states and properties](https://www.w3.org/TR/wai-aria/#states_and_properties) knowledge are required
- These are filled with HTML attributes that are fully supported in JSX and enables to construct fully accessible, highly functional React components


## Other points for consideration

### Setting the language

- Indicate the human language of page text as screen reader software uses this to select the correct voice settings