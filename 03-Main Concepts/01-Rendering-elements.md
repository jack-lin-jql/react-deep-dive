# Rendering elements

- Elements are the smallest building blocks of React apps

```
// An element describes what one wants to see on the screen

const element = <h1>Hello, world</h1>;
```

- Unlike browser DOM elements, React elements are plain objects and are cheap to create
- React DOM takes care of updating the DOM to match the React elements
- Note: there's a distinction between React components and React elements. React elements are what React components are "made of"

## Rendering an element into the DOM

- Say there's a `<div>` somewhere in a HTML file

`<div id="root"></div>`

- This is called a root DOM node since everything inside it will be managed by React DOM
- Applications built with just React usually have a single root DOM node
- If one is integrating React into an existing app, one may have as many isolated root DOM nodes as one would like
- For rendering a React element into a root DOM node, pass both to `ReactDOM.render()`

```
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root));
```

## Updating the rendered element