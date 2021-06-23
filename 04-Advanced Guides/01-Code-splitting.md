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

## import()

- The best way to introduce code-splitting into an app is through the dynamic `import()` syntax

```
// Before
import { add } from "./math";

console.log(add(16, 26));
```

```
// After
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

- When Webpack come across this syntax, it automatically starts code-splitting the app
- If one is using CRA, this is already configured and one can start using it immediately. `Next.js` also has out of the box support
- If one's setting up Webpack themselves, read Webpack's documentation on code splitting 
- When using Babel, one needs to make sure the Babel can parse the dynamic import but is not transforming it. One will need `@babel/plugin-syntax-dynamic-import`

## React.lazy

- Note: React.lazy and Suspense aren't available for server-side rendering yet. If one wants to do code-splitting in a server rendered app, it's recommended to use [Loadable Components](https://github.com/gregberge/loadable-components)
- React.lazy allows one to render a dynamic import as a regular component

```
// Before
import OtherComponent from "./OtherComponent";
```

```
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```

- This will automatically load the bundle containing the `OtherComponent` when this component is first rendered
- `React.lazy` takes a function that must call a dynamic `import()`
- This must return a `Promise` which resolves to a module with a `default` export containing a React component
- The lazy component should then be rendered inside a `Suspense` component, allowing us to show some fallback content (such as a loading indicator) while waiting for the lazy component to load

```
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

- The fallback prop accepts any React element that one wants to render when waiting for the component to load
- One can place the `Suspense` component anywhere above the lazy component
- One can even wrap multiple lazy components with a single `Suspense` component

```
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherOtherComponent />
        </section>
      </Suspense>
    </div>
  )
}
```

### Error boundaries

- If other modules fail to load (e.g. due to network failure), it'll trigger an error
- One can handle these errors to show a nice UX and manage recovery with Error Boundaries
- Once an Error Boundary is created, it can be used anywhere above lazy components to display an error state when there's a network error

```
import React, { Suspense } from 'react';
import MyErrorBoundary from './MyErrorBoundary';

const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherOtherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

## Route-based code splitting

- Deciding where in an app to introduce code splitting can be tricky
- One wants to make sure it's chosen at places where it will split bundles evenly, but won't disrupt the UX
- Routes is a good place to start
- Most people on the web are used to page transitions taking some amount of time to load
- It also tends to be re-rendering the entire page at once so users are unlikely to be interacting with other elements on the page at the same time

```
import React, { Suspense, lazy } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';

const Home = lazy(() => import('./routes/Home));
const About = lazy(() => import('./routes/About));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/about" component={About} />
      </Switch>
    </Suspense>
  </Router>
)
```

## Named exports

- `React.lazy` currently only supports default exports
- If the modules one wants to import uses named exports, one can create an intermediate module that reexports it as the default
- This ensures tree shaking keeps working and that one doesn't pull in unused components

```
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;


// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";

// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```