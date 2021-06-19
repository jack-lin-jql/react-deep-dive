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