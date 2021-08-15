# ReactDOMServer

- The `ReactDOMServer` object enables one to render components to static markup, typically used on a Node server

```
// ES modules
import ReactDOMServer from "react-dom/server";

// CommonJS
var ReactDOMServer = require("react-dom/server");
```

## Overview

- The following method can be used in both the server and browser environments:
  - renderToString()
  - renderToStaticMarkup()
- These additional methods depend on a package (stream) that is only available on the server and won't work in the browser
  - renderToNodeStream()
  - renderToStaticNodeStream()

## Reference

### renderToString()

`ReactDOMServer.renderToString(element)`

- Render a React element to its initial HTML
- React will return an HTML string, one can use this method to generate HTML on the server and send the markup down on the initial request for faster page loads and to allow search engine to crawl one's pages for SEO purposes 
- If one calls `ReactDOM.hydrate()` on a node that already has this server-rendered markup, React will preserve it and only attach event handlers, allowing one to have a very performant first-load experience

### renderToStaticMarkup()

`ReactDOMServer.renderToStaticMarkup(element)`

- Similar to `renderToString`, except this doesn't create extra DOM attributes that React uses internally, such as data-reactroot
- This is useful if one wants to use React as a simple static page generator, as stripping away the extra attributes can save some bytes
- If one plans to use React on the client to make the markup interactive, do not use this method. Use renderToString on the server and ReactDOM.hydrate() on the client

### renderToNodeStream()

`ReactDOMServer.renderToNodeStream(element)`

- Render a React element to its initial HTML, returns a Readable stream that outputs an HTML string
- The HTML output by thi stream is exactly equal to what ReactDOMServer.renderToString would return
- One can use this method to generate HTML on the server and send the markup down on the initial request for faster page loads and to allow search engine to crawl your pages for SEO purposes
- If one calls ReactDOM.hydrate() on a node that already had this server-rendered markup, React will preserve it and only attach event listeners, allowing one to have a very performant first-load experience
- Note: Server-only, not available in browser
  - The stream returned from this method will return a byte stream encoded in utf-8

### renderToStaticNodeStream()

`ReactDOMServer.renderToStaticNodeStream(element)`

- Similar to renderToNodeStream, except this doesn't create extra DOM attributes that React uses internally, such as `data-reactroot`
- This is useful for simple static page generator, as it strips away the extra attributes and saves some bytes
- The HTML output by this stream is exactly equal to what `ReactDOMServer.renderToStaticMarkup` would return
- If planning on using React on the client to make the markup interactive, don't use this
- Use renderToNodeStream on the server and ReactDOM.hydrate() on the client
- Note: Server-only, not available in browser
  - The stream returned from this method is encoded in utf-8