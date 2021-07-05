# JSX in depth

- Fundamentally, jSX just provides syntactic sugar for the `React.createElement(component, props, ...children)` function

```
// In JSX
<MyButton color="blue" shadowSize={2}>
  Click me
</MyButton>
```

```
// Compiles to
React.createElement(
  MyButton,
  {color, 'blue', shadowSize: 2},
  'Click Me'
)
```

- One can also use the self-closing form of the tag if no children

`<div className="sidebar" />`

```
// Compiles to
React.creteElement(
  'div',
  {className: 'sidebar'}
)
```

- If one wants to test out how specific JSX is converted into JS, oen can try out the [online Babel compiler](https://babeljs.io/repl/#?presets=react&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA)

## Specifying the React element type

- The first part of a JSX tag determines the type of the React element
- **Capitalized types indicate that the JSX tag is referring to a React component**
- These **tags get compiled into a direct reference to the named variable**, so if one use the JSX `<Foo />` expression, `Foo` must be in scope

### React must be in scope

- Since JSX compiles into calls to `React.createElement`, the `React` library must also always be in scope from your JSX code
- E.g., both of the imports are necessary in this code, even though `React` and `CustomButton` are not directly referenced from JS

```
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null)
  return <CustomButton color='red' />;
}
```

- If one doesn't use a JS bundler and loaded React from a `<script>` tag, it is already in scope as the `React` global

### Using dot notation for JSX type

- One can also refer to a React component using dot-notation from within JSX
- This is convenient if one has a single module that exports many React components
- E.g. `MyComponents.DatePicker` is a component, one can use it directly from JSX with:

```
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} date picker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponent.DatePicker color="blue" />;
}
```

### User-defined components must be capitalized

- When an element type starts with a lowercase letter, it refers to a built-in component like `<div>` or `<span>` and results in a string `'div'` or `'span'` passed to `React.createElement`
- Types that start with a capital letter like `<Foo />` compile to `React.createElement(Foo)` and correspond to a component defined or imported in the JS file
- It's recommended to name components with a capital letter, if one does have a component that starts with a lowercase letter, assign it to a capitalized variable before using it in JSX

```
// E.g. this code will not run as expected
import React from 'react';

// Wrong this is a component and should have been capitalized
function hello(props) {
  // Correct! This use of <div> is legitimate since div is a valid HTML tag
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Wrong! React thinks <hello /> is an HTML tag since it's not capitalized
  return <hell toWhat="World" />;
}
```

- To fix this, one will rename `hello` to `Hello` and use `<Hello />` when referring to it

```
import React from 'react';

// Correct, this is a component and should be capitalized
function Hello(props) {
  // Correct! This use of <div> is legitimate since div is a valid HTML tag
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {\
  // Correct! React knows <Hello /> is a component because it's capitalized
  return <Hello toWhat="World" />;
}
```

### Choosing the type at runtime

- One cannot use a general expression as the React element type
- If one does want to use a general expression to indicate the type of the element, just assign it to a capitalized variable first
- This often comes up when one wants to render a different component based on a prop

```
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Wrong! JSX type can't be an expression
  return <components[props.storyType] story={props.story} />;
}
```

- To fix, one can assign the type to a capitalized variable first

```
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable
  const SpecificStory = component[props.storyType];
  return <SpecificStory story={props.story} />;
}
```