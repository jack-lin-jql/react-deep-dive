# Lists and keys

- Let's review how one transform lists in JS first
- Followed below, the `map()` function is used to take an array of numbers and double their values. The new array returned by `map()` gets assigned to the variable `doubled` after transformation

```
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
```

- React transformation of arrays into list of elements is nearly identical

## Rendering multiple components

- One can build collections of elements and include them in JSX using curly braces `{}`

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) => <li>{number}</li>);

ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);
```

## Basic list component

- Usually, lists are rendered inside a component

```
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) => <li>{number}</li>);
  return (<ul>{listItems}</ul>);
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

- Upon running this code, one will be given a warning that a key should be provided for the list items
- A `key` is a special string attribute one need to include when creating lists of elements

```
// We can fix as the following
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

## Keys

- Keys help React identify which item have changed, are added, or are removed
- Keys should be given to the element inside the array to give the elements a stable identify

```
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```

- Best way to pick a key is to use a string that uniquely identifies a list item among its siblings. Most often, one would use IDs from the data as keys

```
const todoItems = todos.map((todo) => 
  <li key={todo.id}>
    {todo.text}
  </li>
);
```