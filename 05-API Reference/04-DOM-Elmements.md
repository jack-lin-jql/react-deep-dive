# DOM Elements

- React implements a browser-independent DOM system for performance and cross-browser compatibility
- In React, all DOM properties and attributes (including event handlers) should be camelCased
- E.g. the HTML attribute `tabindex` corresponds to the attribute `tabIndex` in React
- The except is `aria-*` and `data-*` attributes, which should be lowercased
- E.g. one can keep `aria-label` as is

## Differences in attributes

- These are number of attributes that work differently between React and HTML

### checked

- The `checked` attribute is supported by `<input>` components of type `checkbox` or `radio`
- One can use it to set whether the component is checked
- This is useful for building controlled components
- `defaultChecked` is the uncontrolled equivalent, which sets whether the component is checked when it is first mounted

### className

- To specify a CSS class, use the `className` attribute
- This applies to all regular DOM and SVG elements like `<div>`, `<a>` and others
- If one uses React with Web COmponents (which is uncommon), use the `class` attribute instead

### dangerouslySetInnerHTML

- `dangerouslySetInnerHTML` is React's replacement for using `innerHTML` in the browser DOM
- In general, setting HTML from code is risky since it's easy to inadvertently expose uses to XSS attach
- So, one can set HTML directly from React, but one have to type out `dangerouslySetInnerHTML` and pass an object with a `__html` key to remind oneself that it's dangerous

```
function createMarkup() {
  return { __html: 'first &middot; Second'};
}

function MyComponent() {
  return <div dangerouslySetInnerHTML={createMarkup()} />;
}
```

### htmlFor

- Since `for` is a reserved word in JS, React elements use `htmlFor` instead

### onChange

- The onChange event behaves as one would expect it to: whenever a form field is changed, this event is fired
- It's intentional that existing browser behavior isn't used because onChange is a misnomer for its behavior and React relies on this event to handle user input in real time

### selected

- If one wants to mark an `<option>` as selected, reference the value of that option in the `value` of its `<select>` instead

### style

- Note: Using the style attribute as the primary means of styling is generally NOT recommended
- In most cases, `className` should be used to reference classes defined in an external CSS stylesheet
- `style` is most often used in React applications to add dynamically-computed styles at render time
- The style attribute accepts a JS object with camelCased properties rather than a CSS string
- This is consistent with the DOM style JS property, is more efficient and prevents XSS security holes

```
const divStyle = {
  color: 'blue',
  backgroundImage: 'url(' + imgUrl + ')',
};

function HelloWorldComponent() {
  return <div style={divStyle}>Hello World!</div>;
}
```

- Note that styles aren't autoprefixed, to support older browsers, one needs to supply corresponding style properties

```
const divStyle = {
  WebkitTransition: 'all', // note the capital W here
  msTransition: 'all', 'ms' is the only lowercase vendor prefix
};

function ComponentWithTransition() {
  return <div style={divStyle}>This should work cross-browser</div>;
}
```

- Style keys are camelCased in order to be consistent with accessing the properties on DOM nodes from JS (e.g. `node.style.backgroundImage`)
- Vendor prefixes other than ms should begin with a capital letter
- This is why WebkitTransition has an uppercase "W"
- React will auto append a "px" suffix to certain numeric inline style properties
- If one wants to use units other than "px", specify the value as a string with the desired unit

```
// Result style: 10px
<div style={{ height: 10 }}>
  Hello World!
</div>

// Result style: '10%'
<div style={{ height: '10%' }}>
  Hello World!
</div>
```

- Not all style properties are converted to pixel strings though
- Certain ones remain unitless (e.g. zoom, order, flex)
