# ReactDOM

- If one loads React from a `<script>` tag, these top-level APIs are available on the ReactDOM global
- If one uses ES6 with npm, one can write `import ReactDOM from "react-dom"`
- If using ES6 with npm, one can write `var ReactDOM = require('react-dom')`

## Overview

- The `react-dom` package provides DOM-specific methods that can be used at the top level of an app and as an escape hatch to get outside of the React model if one needs to
- Most of components shouldn't need to use this module

- render()
- hydrate()
- unmountComponentAtNode()
- findDOMNode()
- createPortal()

## Browser support

- React supports all popular browsers, including IE9+ although some polyfills are required for older browsers such as IE 9 and IE10
- Note: Older browsers that don't support ES5 methods aren't supported, but one may find apps do work in older browsers if polyfills such as es5-shim and es5-sham are included in the page

## Reference

### render()

`ReactDOM.render(element, container[, callback])`

- Render a React element into the DOM in the supplied `container` and return a reference to the component (or return `null` for stateless components)
- If the React element was previously rendered into `container`, this will perform an update on ti and only mutate the DOM as necessary to reflect the latest React element
- If the optional callback is provided, it'll be executed after the component is **rendered or updated**
- Note: `ReactDOM.render()` controls the contents of the container node one passes in
  - Any existing DOM element inside are replaced when first called. Later calls use React's DOM diffing algorithm for efficient updates
  - `ReactDOM.render()` does not modify the container node (only modifies the children of the container)
  - It may be possible to insert a component to an existing DOM node without overwriting the existing children
  - `ReactDOM.render()` currently returns a reference to the root `ReactComponent` instance. However, using this return value is legacy and should be avoided because future versions of React may render components asynchronously in some cases
  - If one needs a reference to the root `ReactComponent` instance, the preferred solution is to attach a `callback ref` to the root element
  - Using `ReactDOM.render()` to hydrate a server-rendered container is deprecated and will be removed in React 17. Use `hydrate()` instead

### hydrate()

`ReactDOM.hydrate(element, container[, callback])`

- Same as `render()`, but is used to hydrate a container whose HTML contents were rendered by ReactDOMServer
- React will attempt to attach event listeners to the existing markup

- React expects that the rendered content is identical between the server and the client
- It can patch up differences in the text content, but one should treat mismatches as bugs and fix them
- In dev mode, React warns about mismatch during hydration
- There are no guarantees that attribute differences will be patched up in case of mismatches
- This is important for performance reasons because in most apps, mismatches are rare, and so validating all markup would be prohibitively expensive

- If a single element's attribute or text content is unavoidably different between the server and the client (e.g. timestamp), one may silence the warning by adding `suppressHydrationWarning={true}` to the element. It only works one level deep and is intended to be an escape hatch. Don't overuse it
- Unless it's text content, React still won't attempt to patch it up, so it may remain inconsistent until future updates

- If one intentionally need to render something different on the server and the client, one can do a two-pass rendering
- Components that render something different on the client can read a state variable like `this.state.isClient`, which one can set to `true` in `componentDidMount()`
- This way the initial render pass will render the same content as server, avoiding mismatches, but and additional pass will happen synchronously right after hydration
- Note that this approach will make components slower since they have to render twice, use with caution

- Remember to be mindful of UX on slow connections
- The JS code may load significantly later than the initial HTML render, so if one renders something different in the client-only pass, the transition can be jarring
- However, if executed well, it may be beneficial to render a "shell" of the application on the server, and only show some of the extra widgets on the client

### unmountComponentAtNode()

`ReactDOM.unmountComponentAtNode(container)`

- Remove a mounted React component from the DOM and clean up its event handlers and state
- If no component was mounted in the container, calling this function does nothing
- Returns `true` if a component was unmounted and `false` if there was no component to unmount

### findDOMNode()

- Note: `findDOMNode` is an escape hatch used to access the underlying DOM node
  - In most cases, use of this escape hatch is discouraged because it pierces the component abstraction, so it has been deprecated in StrictMode

`ReactDOM.findDOMNode(component)`

- If this component has been mounted into the DOM, this returns the corresponding native browser DOM element
- This method is useful for reading values out of the DOM, such as form field values and performing DOM measurements
- In most cases, one can attach a ref to the DOM node and avoid using findDOMNode at all
- When a component renders to `null` or `false`, `findDOMNode` returns `null`
- When a component renders to a string, `findDOMNode` returns a text DOM node containing that value
- As of React 16, a component may return a fragment with multiple children, in which case `findDOMNode` will return the DOM node corresponding to the **first non-empty child**
- Note: `findDOMNode` only works on mounted components (that is, components that have been placed in the DOM). If one tries to call this on a component that has not been mounted yet (like calling `findDOMNode()` in `render()` on a component that has yet to be created) an exception will be thrown
- `findDOMNode` cannot be used on function components

### createPortal()

`ReactDOM.createPortal(child, container)`

- Creates a portal which provides a way to render children into a DOM node that exists outside the hierarchy of the DOM component