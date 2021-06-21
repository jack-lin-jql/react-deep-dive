# Code-splitting

## Bundling

- Most React apps will have their files "bundled" using tools like Webpack, Rollup, or Browserify
- Bundling is the process of following imported file and merging them into a single file: a "bundle" 
- This bundle can then be included on a webpage to load an entire app at once

- E.g.
  - App:

```
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```

```
// math.js
export function add(a, b) {
  return a + b;
}
```

  - Bundle:

```
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```

- Note: real bundles will look a lot different
- If one is using CRA, Next.js, Gatsby, or a similar tool, Webpack would have been setup out of the box to bundle the application
- If not, bundling needs to be done manually with infamously Webpack

## Code Splitting

- Bundling is great, but as an app grows, the bundle will grow too
- Especially if large 3rd party libraries are included
- One needs to keep an eye on the code that's being included in the bundle so one doesn't accidentally make it so large that it takes a long time to load
- To avoid ending up with a large bundle, we can get ahead of the problem by splitting a bundle
- Code-splitting is a feature supported by bundlers mentioned before (e.g. Webpack) which can **create multiple bundles that can be dynamically loaded at runtime**
- Code-splitting one's app can help one lazy-load just the things that are currently needed by the user, drastically improving the performance of an app
- While one hasn't reduced the overall amount of code in the app, one's avoided loading code that the user many never need, and reduced the amount of code needed during the initial load

