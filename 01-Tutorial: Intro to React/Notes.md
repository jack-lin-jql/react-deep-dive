# What are we building?

- Interactive tic-tac-toe game

# What is React?

- A declarative, efficient, and flexible JS library for building UIs. It let's one compose complex UIs from small and isolated pieces of code called "components"

### `React.Component` subclasses

```
class ShoppingList extends React.Component {
  render() {
    return (
      <div className="shopping-list">
        <h1>Shopping List for {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    )
  }
}

// Example usage: <ShoppingList name="Mark" />
```

- Components are used to tell React what to show on the screen. When data changes, React will efficiently update and re-render the components
- Here, Shopping List is a **React component class**, or **React component type**
- A component takes in parameters called `props` and returns a hierarchy of views to display via the `render` method. This method returns a description of what to show on the screen where then React takes the description and displays the result
- `render` in particular returns a **React element**, a lightweight description of what to render
- Most React developers use the JSX syntax where `<div />` syntax gets transformed to `React.createElement('div')` at build time
- The example above is equivalent to:

```
return React.createElement('div', { className: 'shopping-list' },
  React.createElement('h1', /* ... h1 children ... */),
  React.createElement('ul', /* ... ul children ... */)
);
```

- JSX comes with the full power of JS and one can put any JS expression within braces inside JSX
- Each React element is a JS object that one can store in a variable or pass around in a program
- Each React component is **encapsulated and can operate independently**; allowing complex UIs from simple components

# Notes

- To collect data from multiple children or have two child components communicate with each other, one need to declare and the shared state in their parent component instead
- The parent component can pass the state back down to the children through props; this keeps the child components in sync with each other and with the parent component
- It's conventional in React to use `on[Event]` names for props which represent events and `handle[Event]` for the methods which handle the events
- Controlled components: when a component's presentation is controlled by a parent component

## Why immutability is important?

- There are generally two approaches to changing data. First being to mutate the data by directly changing the data's values. Second being to replace the data with a new copy which has the desired changes

### Data change with mutation

```
var player = { score: 1, name: 'Jeff' };
player.score = 2;

// Now player is { score: 2, name: 'Jeff }
```

### Data change without mutation

```
var player = { score: 1, name: 'Jeff' };
var newPlayer = Object.assign({}, player, { score: 2 });
// Now player is unchanged, but newPlayer is { score: 2, name: 'Jeff' }

// Or if you are using object spread syntax, one can write
var newPlayer = { ...player, score: 2 };
```

- By not mutating directly, several benefits are gained

### Complex feature become simple

- Immutability makes complex feature easier to implement
- For example, the ability to undo and redo certain actions is a common requirement in applications and by avoiding direct data mutation, it allows us to keep previous states of the application to be reused later

### Detecting changes

- Change detection is in mutable objects is difficult since they're modified directly. The detection would require the mutable object to be compared to previous copies of itself and the entire object tree to be traversed
- As for immutable objects, if a different immutable object is being referenced than the previous one, then the object has changed

### Determining when to re-render in React

- Immutability also helps with building pure components since immutable data can easily determine if changes have been made, which helps to determine if a component requires re-rendering

## Function components

- Function components are a simpler way to write components that only contain a render method and don't have their own state
- Instead of a class that extends React.Component, one can write a function that takes props as input and returns what should be rendered

## Picking a key

- When rendering a list, React stores some information about each rendered list item. When this list updates, React needs to determine what has changed
- For example:
```
// From
<li>Alexa: 7 tasks left</li>
<li>Ben: 5 tasks left</li>

// To
<li>Ben: 9 tasks left</li>
<li>Claudia: 8 tasks left</li>
<li>Alexa: 5 tasks left</li>
```
- Here, in addition to updated counts, Alexa and Ben were swapped. We need to specify a key for each list item to differentiate each from its siblings
```
<li key={user.id}>{user.name}: {user.taskCount} tasks left</li>
```
- Now, when a list is re-rendered, React takes each list item's key and searches the previous list's items for a matching key
  - If the current list has a new key, React creates a component
  - If the current list is missing a key that existed previously, React destroys the previous component
  - If two keys match, the corresponding component is moved
- Keys tell React about the identity of each component's key changes, the component will be destroyed and recreated with a new state
- `key` is a special reserved property in React. When an element is created, React extracts the `key` property and stores the key directly on the returned element
- `key` cannot be referenced using `this.props.key` even though it may look like it belongs in `props`
- It's strongly recommended that one assign proper keys when building dynamic lists
- If no key is specified, React throws a warning and uses array index as a key by default. This is problematic when reordering or inserting/removing list items
- Explicitly passing `key={index_value}` silences the warning, but the same reordering problems exist
- Keys do not need to be globally unique, but they need to be unique between components and their siblings