# Glossary of React terms

## SPA

- A SPA is an application that loads a single HTML page and all the necessary assets (such as JS and CSS) required for the application to run
- Any interaction with the page or subsequent pages do not require a round trip to the server which means the page isn't reloaded
- Though one may build a SPA in React, it's not a requirement
- React can also be used for enhancing small parts of existing websites with additional interactivity
- Code written in React can coexist with markup rendered on the server by something like PHP, or with other client-side libraries

## ES6, ES2015, ES2016, etc

- These all refer to the most recent version of ECMAScript Language Spec standard which the JS language is an implementation of
- The ES6 version (also known as ES2015) includes many additions to the previous version such as: arrow functions, classes, template literals, `let` and `const` statements

## Compilers

- A JS compiler takes JS code, transforms it and returns JS code in a different format
- The most common use case it to take ES6 syntax and transform into syntax that older browsers are capable of interpreting
- Babel is the compiler most commonly used with React

## Bundlers

- Bundlers take JS and CSS code written as separate modules (often hundreds of them) and combine them together into a few files better optimized for the browsers
- Some bundlers commonly used in React applications include Webpack and Browserify

## Package managers

- These are tools that allow one to manage dependencies in a project
- `npm` and `yarn` are 2 package managers commonly used and both are clients of the same npm package registry

## CDN

- Content Delivery Network - delivers cached static content from a network of servers across the globe

## JSX

- Syntax extension to JS, similar to a template language, but it has full power of JS
- Compiled into `React.createElement()` calls which return plain JS objects called "React elements"`
- React DOM uses camelCase property naming convention instead of HTML attribute names
  - E.g. `tabindex` -> `tabIndex` in JSX
  - `class` -> `className` since `class` is reserved in JS

```
const name = "Clementine";
ReactDOM.render(
  <h1 className="hello">My name is {name}!</h1>,
  document.getElementById("roo")
);
```

## Elements

- React elements are the building block of React applications
- One might confuse elements with a more widely known concept of "components"
- An element describes what one want to see on the screen and React elements are immutable 

`const element = <h1>Hello, world</h1>;`

- Typically, elements aren't used directly, but get returned from components

## Components

- React components are small, reusable pieces of code that return a React element to be rendered to the page
- The simplest version of React components is a plain JS function that returns a React element

```
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

```
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

- Components can be broken down into distinct pieces of functionality and used within other components
- Components can return other components, arrays, strings and numbers
- A good rule of thumb is if a part of UI is used several times or is complex enough on its own, it's a good candidate to be a reusable component
- Component names should also always start with a capital letter (`<Wrapper />` and not `<wrapper />`)

## props

- `props` are inputs to a React component, they are data passed down from a parent component to a child component
- Remember that props are readonly and should not be modified in any way

```
// Wrong!
props.number = 42;
```
- If modification is required, use state instead

## props.children

- Available on every component, it contains the content between the opening and closing tags of a component

`<Welcome>Hello world!</Welcome>`

- The string `Hello world!` is available in `props.children` in the `Welcome` component

```
function Welcome(props) {
  return <p>{props.children}</p>;
}
```

```
class Welcome extends React.Component {
  render() {
    return <p>{this.props.children}</p>;
  }
}
```

## state

- A component needs `state` when some data associated with it changed over time
- E.g. a Checkbox component might need `isChecked` in its state
- The most important difference between `state` and `props` is that `props` are passed from a parent component, but `state` is manged by the component itself
- A component cannot change its `props` but it can change its `state`
- For each particular piece of changing data, these should be just one component that "owns" it in its state
- Don't try to synchronize state of two different components, instead, lift it up to their closest shared ancestor and pass it down as props to both of them

## Lifecycle methods

- Custom functionality that gets executed during the different phases of a component
- There are method available when the component gets created and inserted into the DOM (mounting), when the component updates, and when the components gets unmounted or removed from the DOM

## Controlled vs uncontrolled components

- React has two different approaches to dealing with form inputs
- An input form element whose value is controlled by React is called a controlled component
  - When a user enters data into a controlled component, a change event handler is triggered and one's code decides whether the input is valid (by re-rendering with the updated value)
  - If one doesn't re-render then the form element will remain unchanged
- An uncontrolled component works like form elements do outside of React
  - When a user inputs data into a form field (an input box, dropdown, etc) the updated information is reflected without React needing to do anything
  - However this also means that one can't force the field to have a certain value
- Most cases, one should use controlled components

## Keys

- A "key" is a special string attribute one needs to include when creating arrays of elements
- It helps React identify which items have changed, are added or are removed
- Keys should be given to the elements inside an array to give the elements a stable identity
- Keys only need to be unique among sibling elements in the same array. NO need to be unique across the whole application or even a single component
- Don't pass something like `Math.random()` to keys since that's not a stable identity! When it re-renders, a new random number is generated
- Ideally, keys should correspond to unique and stable identifiers coming from one's data, such as `post.id`

## Refs

- `ref` attribute can be an object created by `React.createRef()` or a callback function
- When the `ref` attribute is a callback function, the function receives the underlying DOM element or class instance (depending on the type of element) as its argument, allowing one to get direct access to the DOM element or component instance


## Events

- React event handling has some syntactic differences:
  - React event handlers are named using camelCase instead of lowercase
  - With JSX, one passes a function as the event handler rather than a string

## Reconciliation

- When a component's prop or state change, React decides whether the actual DOM update is necessary by comparing the newly returned element with the previously rendered one
- When they aren't equal, React will update the DOM