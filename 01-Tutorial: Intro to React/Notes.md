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