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

## Props in JSX

- There are several different ways to specify props in JSX
 

### JS expression as props

- One can pass any JS expression as a prop, but surrounding with `{}`

`<MyComponent foo={1 + 2 + 3 + 4} />`

- For `MyComponent` the value of `props.foo` will be 10 since the expression gets evaluated
- `if` statements and `for` loops are not expressions in JS, so they can't be used in JSX directly
- Instead, one needs to put these in the surrounding code

```
function NumberDescribe(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }

  return <div>{props.number} is an {description} number</div>;
}
```

### String literals

- One can pass a string literal as a prop

```
// Following is equivalent
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

- When one passes a string literal, its value is HTML-unescaped, so the following are equivalent

```
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

- This behavior is usually irrelevant

### Props default to "true"

- If one passes no value for a prop, it defaults to `true`

```
// Following are equivalent
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

- In general, we **don't recommend not passing a value for a prop**, because it can be confused with the ES6 object shorthand `{foo}` which is short for `{foo: foo}` rather than `{foo: true}`
- This behavior is there so that it matches the behavior of HTML

### Spread attributes

- If one already have `props` as an object and want to pass it in JSX, one can use `...` as a spread operator to pass the whole props object

```
// The following is equivalent
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

- One can also pick specific props that a component will consume while all other props using the spread operator

```
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";

  return <button className={className} {...other} />;
}

const App = () => {
  return (
    <div>
      <Button kind="primary", onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
}
```

- In the example above, the `kind` prop is safely consumed and is not passed on to the `<button>` element in the DOM
- All other props are passed via the `...other` object making this component really flexible
- Spread attributes can be useful but they also make it easy to pass unnecessary props to components that don't care about them or pass invalid HTML attributes to the DOM. Use this syntax sparingly

## Children in JSX

- In JSX expressions that contain both an opening tag and a closing tag, the content between those tags is passed as a special prop: `props.children`
- There are several different ways to pass children

### String literals

- One can put a string between the opening and closing tags and `props.children` will just be that string. Useful for many of the built-in HTML elements

`<MyComponent>Hello world!</MyComponent>`

- This is valid JSX and `props.children` in `MyComponent` will simply be the string `"Hello world!"`
- HTML Is unescaped, so one can generally write JSX just like one would write HTML in this way

`<div>This is a valid HTML &amp; JSX at the same time.</div>`

- JSX removes whitespace at the beginning and the ending of a line
- It also removes blank lines. New lines adjacent to tags are removed; new lines that occur in the middle of string literals are condensed into a single space

```
// The following all render to the same thing
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### JSX children

- One can provide more JSX elements as the children, which is useful for displaying nested components

```
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

- One can mix together different types of children, so one can use string literals together with JSX children
- This is another way in which JSX is like HTML, so that this is both valid JSX and valid HTML:

```
<div>
  Here is a list:
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
```

- A React component can also return an array of elements

```
render() {
  // No need to wrap list item in an extra element!
  return [
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ]
}
```