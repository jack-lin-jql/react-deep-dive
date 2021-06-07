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

- React elements are immutable, which means that once an element is created, it cannot change its children or attributes
- Think of an element like a single frame in a movie, it represents the UI at a certain point in time
- So far, the only way to update the UI is to create a new element and pass it to `ReactDOM.render()`

```
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

- This calls `ReactDOM.render()` every second from a `setInterval()` callback
- In practice, most React apps only call `ReactDOM.render()` once and such code gets encapsulated into stateful components

## React only updates what's necessary

- React DOM compares the element and its children to the previous one and only applies the DOM updates necessary to bring the DOM to the desired state
- For example, in the previous example, even though the entire element describe the whole UI tree are recreated upon every tick, only the text node whose content have changed gets updated by React DOM