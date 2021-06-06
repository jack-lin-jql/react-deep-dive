# Introducing JSX

`const element = <h1>Hello, world!</h1>;`

- The tag syntax there is neither a string nor HTML
- It is JSX and it's a **syntax extension** to JS
- JSX produces React elements

## Why JSX?

- React embraces the fact that the rendering logic is inherently coupled with other UI logic. E.g. how events are handled, how the state changes over time, and how the data is prepared for display
- So instead of putting markup and logic in separate files, React separates concerns with loosely coupled units called components that contain both
- React doesn't require using JSX, but it allows React to show more useful error and warning messages while providing useful visual aids

## Embedding expression in JSX

```
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

- One can put any valid JS expression inside the curly braces in JSX, e.g. `2 + 2`, `user.firstName`, or `formatName(user)`
- Embedding the result of a JS function into an `<h1>` element:

```
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

## JSX is an expression too

- After compilation, JSX expressions become regular JS function calls and evaluate to JS objects
- This means one can use JSX inside of `if` states and `for` loops, assign it to variables, accept it as arguments, and even return from functions

```
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stringer.</h1>;
}
```

## Specifying attributes with JSX

- One may use quotes to specify string literal as attributes

`const element = <div tabIndex="0" />`

- One may also use curly braces to embed a JS expression in an attribute

`const element = <img src={user.avatarUrl} />`

- One should either use quotes for string values or curly braces for expression, but not both in the same attribute
- Since JSX is closer to JS than to HTML, React DOM uses camelCase property naming convention instead of HTML attribute names
- E.g. `class` becomes `className` in JSX, and `tabindex` becomes `tabIndex`

## Specifying children with JSX

- If a tag is empty, it can closed immediately with `/>` like XML
`const element = <img src={user.avatarUrl} />;`
- JSX tags may contain children

```
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

## JSX prevents injection attacks

- It's safe to embed user input in JSX as such

```
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

- By default, React DOM escapes any values embedded in JSX before rendering them
- This ensures one can never inject anything that's not explicitly written in one's application
- Everything is converted to a string before being rendered, which helps in preventing XSS attacks

## JSX represents objects

- Babel compiles JSX down to React.createElement() calls
- For example:

```
// JSX
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);

// JS
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

- `React.createElement()` performs a few checks to help one write bug-free code and essentially creates an object like this:

```
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```

- These objects are called **"React elements"** which one can think of them as descriptions of what to be seen on the screen
- React reads these objects and uses them to construct the DOM and keep it up to date