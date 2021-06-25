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

- 