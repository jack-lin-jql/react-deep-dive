# Web components

- React and Web Components are built to solve different problems
- Web Components provide strong encapsulation for reusable components, while React provides a declarative library that keep the DOM in sync with one's data
- The two goals are complementary
- As a developer, one is free to use React in Web Components, or to use Web Component in React, or both
- Most people who use React don't use Web Components, but one may want to, especially if one is using third-party UI components that are written using Web Components

## Using web components in React

```
class HelloMessage extends React.Component {
  render() {
    return <div>Hello <x-search>{this.props.name}</x-search>!</div>;
  }
}
```

- Note: Web Components often expose an imperative API. E.g. a video Web Component might expose play() and pause() functions. To access the imperative APIs of a WC, one will need to use a ref to interact with the DOM node directly. If one is using third-party WCs, the best solution is to write a React component that behaves as a wrapper for the WC.
- Note: Events omitted by a WC may not properly propagate through a React render tree. One will need to manually attach event handlers to handle these events within their React components
- One common confusion is that WX use "class" instead of "className"

```
function BrickFlipbox() {
  return (
    <brick-flipbox class="demo">
      <div>front</div>
      <div>back</div>
    </brick>
  );
}
```

## Using React in WCs

```
class XSearch extends HTMLElements {
  connectedCallback() {
    const mountPoint = document.createElement('span');
    this.attachShadow({ mode: 'open' }).appendChild(mountPoint);

    const name = this.getAttribute('name');
    const url = 'https://www.goggle.com/search?q=' + encodeURIComponent(name);
    ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
  }
}

customElements.define('x-search', XSearch);
```

- Note: this code won't work if one transform classes with Babel. Include the `custom-elements-es5-adapter` before one loads WCs to fix this issue