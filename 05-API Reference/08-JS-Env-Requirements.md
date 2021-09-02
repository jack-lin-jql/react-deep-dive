# JavaScript environment requirements

- React 16 depends on the collection types `Map` and `Set`
- If one supports older browsers and devices which may not yet provide these natively (e.g. IE < 11) or which have non-compliant implementations, consider including a global polyfill in the bundled application such as core-js
- A polyfilled env for React 16 using core-js to support older browsers might look like:

```
import 'core-js/es/map';
import 'core-js/es/set';

import React from "react";
import ReactDOM from "react-dom";

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById("root")
);
```

- React also depends on `requestAnimationFrame` (even in test environments)
- One can use the `raf` package to shim `requestAnimationFrame`

`import "raf/polyfill";`