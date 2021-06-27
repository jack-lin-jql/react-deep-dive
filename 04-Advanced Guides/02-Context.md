# Context

- Context provides a way to pass data through the component tree without having to pass props down manually at every level
- Typically, in a React app, data is passed top-down (parent to child) via props, but such usage can be cumbersome for certain types of props (e.g. locale preference, UI theme) where they may be required by many components within an application
- Context provides a way to share values like these between components without having to explicitly pass a prop through every level of the tree

## When to use context 

- Context is designed to share data that can be considered "global" for a tree of React components, such as the current authenticated user, theme, or preferred language
- Below, we manually thread through a "theme" prop in order to style the Button component

```
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  // The Toolbar component must take an extra "theme" prop and pass it to the ThemeButton. This can become painful if every single button in the app needs to know the theme because it would have to be passed through all components

  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
```

- Using context, one can avoid passing props through intermediate elements

```
// Context lets us pass a value deep into the component tree without explicitly threading it through every component (prop drilling).
// Creating a context for the current theme (with "light" as the default).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Use a Provider to pass the current theme to the tree below. Any component can read it, no matter how deep it is. In this example, we're passing "dark" as the current value.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// A component in the middle doesn't have to pass the theme down explicitly anymore
function Toolbar() {
  return (
    <div>
      <ThemeButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // Assign a contextType to read the current theme context. React will find find the closest theme Provider above and use its value
  // In this example, the current theme is "dark"
  static contextType = ThemeContext;

  render() {
    return <Button theme={this.context} />;
  }
}
```

## Before one uses context

- Context is primarily used when some data needs to be accessible by many components at different nesting levels. Apply it sparingly as it makes component reuse more difficult
- If one only wants to avoid passing some props through many levels, component composition is often a simpler solution than context
- E.g. consider `Page` component that passes a `user` and `avatarSize` prop several levels down so that deeply nested `Link` and `Avatar` components can read it

```
<Page user={user} avatarSize={avatarSize} />
// renders
<PageLayout user={user} avatarSize={avatarSize} />
//renders
<NavigationBar user={user} avatarSize={avatarSize} />
// renders
<Link href={user.permalink}>
  <Avatar user={user} size={avatarSize} />
</Link>
```
- It may feel redundant to pass down the two props all the way through so many levels if in the end only `Avatar` is using it
- It's also annoying whenever the `Avartar` component needs more props from the top, one will have to add them at all the intermediate levels too
- To solve this without context is to pass down the `Avatar` component itself so that the intermediate components don't need to know about the `user` or `avatarSize` props

```
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

// Now
<Page user={user} avatarSize={avatarSize} />
// renders
<PageLayout userLink={..} />
//renders
<NavigationBar userLink={..} />
// renders
{props.userLink}
```

- This change allows only the top-most `Page` component to know about the `Link` and `Avatar` components' use of `user` and `avatarSize`
- This inversion of control can make one's code cleaner in many cases by reducing the amount of props one needs to pass through the application and giving more control to the root component
- However, this isn't the right choice in every case; moving more complexity higher in the tree makes those higher-level component more complicated and forces the lower-level components to be more flexible than one may want
- One is not limited to a single child for a component, as one may pass multiple children or even have multiple separate slots for children

```
function Page(props) {
  const user = props.user;
  const content = <Feed user={user} />;
  const topBar = (
    <NavigationBar>
      <Link href={user.permalink}>
        <Avatar user={user} size={props.avatarSize} />
      </Link>
    </NavigationBar>
  );
  return (
    <PageLayout
      topBar={topBar}
      content={content}
    >
  );
}
```

- This pattern is sufficient for many cases when one needs to decouple a child from its immediate parents
- One can take it even further with **render props** (essentially callback rendering through a prop) if the child needs to communicate with the parent before rendering
- However, sometimes the same data needs to be accessible by many components in the tree, and at different nesting levels
- Context lets one "broadcast" such data, and changes to it, to all components below
- Common examples are to use context for locale, theme, or data cache

## API

### React.createContext

`const MyContext = React.createContext(defaultValue);`

- Creates a Context object
- When React renders a component that subscribes to this Context object it will read the current context value form the **closest match `Provider` above it in the tree**
- The `defaultValue` argument is **only used when a component does not have a matching Provider above it in the tree**
- This default value can be helpful for testing components in isolation without wrapping them
- Note: passing `undefined` as a Provider value does not cause consuming component to use `defaultValue`


### Context.Provider

`<MyContext.Provider value={/* some value */} />`

- Every Context object comes with a Provider React component that **allows consuming components to subscribe to context changes**
- The Provider component accepts a `value` prop to be passed to consuming components that are descendants of this Provider
- One Provider can be connected to many consumers (one to many)
- Provider can be nested to override values deeper within the tree
- All consumers that are descendant of a Provider will re-render whenever the Provider's `value` prop changes
- The propagation from Provider to its descendant consumer (including `.contextType` and `useContext`) is **not subject to** the `shouldComponentUpdate` method, so the consumer is updated even when an ancestor component skips an update
- Changes are determined by comparing the new and old values using the same algo as `Object.is`
- Note: The way changes are determined can cause some issues when passing object as `value`

### Class.contextType

```
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
```

- The `contextType` property on a class can be assigned a Context object created by `React.createContext()`
- Using this property lets one consume the nearest current value of that COntext type using `this.context`
- One can reference this in any of the lifecycle methods including the render function
- Note: one can only subscribe to a single context using this API. If one needs to more, I'm guessing `useContext` might be needed
- Note: If one is using the experimental public class field syntax then they can use a static class field to initialize the `contextType`

```
class MyClass extends React.Component {
  static contextType = MyContext;

  render() {
    let value = this.context;
    /* render something based on the value */
  }
}
```

### Context.Consumer

```
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```

- A React component that subscribes to context changes
- Using this component lets one subscribe to a context within a `function component`
- This requires a function as a child (render props concept). The function receives the current context value and returns a React node
- The `value` argument passed to the function will be equal to the `value` prop of the closes Provider for this context above in the tree
- If there's no Provider for this context value, the `value` argument will be equal to the `defaultValue` that was passed to `createContext()`

### Context.displayName

- Context object accepts a `displayName` string property
- React DevTools uses this string to determine what to display for the context

```
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" in DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" in DevTools
```

## Examples

### Dynamic Context

```
// theme-context.js

export const themes = {
  light: {
    foreground: '...',
    background: '...',
  },
  dark: {
    foreground: '...',
    background: '...',
  },
};

export const ThemeContext = React.createContext(
  theme.dark // default value
);
```

```
// themed-button.js
import { ThemeContext } from "./theme-context";

class ThemedButton extends React.Component {
  render() {
    let props = this.props;
    let theme = this.context;
    return (
      <button {...props} style={{ backgroundColor: theme.background }} />
    );
  }
}

ThemedButton.contextType = ThemeContext;

export default ThemedButton;
```

```
// app.js
import {ThemeContext, themes} from "./theme-context";
import ThemedButton from "./themed-button";

// An intermediate component that uses the ThemedButton
function Toolbar(props) {
  return (
    <ThemedButton onClick={props.changeTheme}>
      Change Theme
    </ThemButton>
  );
}

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      theme: themes.light,
    };

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === theme.dark
            ? themes.light : themes.dark,
      }));
    };
  }

  render() {
    // The ThemedButton button inside the ThemeProvider uses the them from state while one outside uses the default dark theme
    return (
      <Page>
        <ThemeContext.Provider value={this.state.theme}>
          <Toolbar changeTheme={this.toggleTheme} />
        </ThemeContext.Provider>
        <Section>
          <ThemedButton />
        </Section>
      </Page>
    );
  }
}

ReactDOM.render(<App />, document.root);
```

### Updating context from a nested component

- It's often necessary to update the context from a component that is nested somewhere deeply in the component tree
- In this case, one can pass a function down through the context to allow consumers to update the context

```
// theme-context.js
// Make sure the shape of the default value passed to createContext matches the shape that the consumer expects!
export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});
```

```
// theme-toggle-button.js
import { ThemeContext } from './theme-context';

function ThemeTogglerButton() {
  // The Theme Toggler Button receives not only the theme but also a toggleTheme function from the context
  return (
    <ThemeContext.Consumer>
      {({theme, toggleTheme}) => (
        <button
          onClick={toggleTheme}
          style={{ backgroundColor: theme.background }}
        >
          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

export default ThemeTogglerButton;
```

```
// app.js
import { ThemeContext, themes } from './theme-context';
import ThemeTogglerButton from './theme-toggler.button';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === themes.dark
            ? themes.light
            : themes.dark,
      }));l
    };

    // State also contains the updater function so it will be passed down into the context provider
    this.state = {
      theme: themes.light,
      toggleTheme: this.toggleTheme,
    };
  }

  render() {
    // The entire state is passed to the provider
    return (
      <ThemeContext.Provider value={this.state}>
        <Content />
      </ThemeContext.Provider>
    );
  }
}

function Content() {
  return (
    <div>
      <ThemeTogglerButton />
    </div>
  );
}

ReactDOM.render(<App />, document.root);
```

### Consuming multiple contexts

- To keep context re-rendering fast, React needs to make each context consumer a separate node in the tree

```
// Theme context default to light theme
const ThemeContext = React.createContext('light');

// Signed-in user context
const UserContext = Rect.createContext({
  name: 'Guest',
});

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // App component provides initial context values
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// A component may consume multiple contexts
function Content() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  )
}
```

- If 2 or more context values are often used together, one might want to consider creating their own render prop component that provides both

## Caveats

- Context uses reference identity to determine when to re-render, there're some gotchas that could trigger unintentional renders in consumers when a provider's parent re-renders 
- E.g. the code below will re-render all consumers every time the Provider re-renders because a new object is always created for `value

```
class App extends React.Component {
  render() {
    return (
      <MyContext.Provider value=({ something: 'something}}>
        <Toolbar />
      </MyContext.Provider>
    );
  }
}
```

- To get around this, life the value into parent's state

```
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: { something: 'something' },
    };
  }

  render() {
    return (
      <MyContext.Consumer value={this.state.value}>
        <Toolbar />
      </MyContext.Consumer>
    )
  }
}
```