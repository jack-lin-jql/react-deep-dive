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

- When no stable IDs are provided for rendered items, one may use the item index as a key as a **last resort**

```
const todoItems = todos.map(todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

- It is not recommended using indices for keys if the order of items may change
- It can negatively impact performance an may cause issues with component state
- If an explicit key to list items isn't assigned then React will default to using indices as keys

## Extracting components with keys

- Keys only make sense in the context of the surrounding array
- E.g. if one extracts a `ListItem` component, they should keep the key on the `<ListItem />` elements in the array rather than on the `<li>` element in the `ListItem` itself

```
// Incorrect key usage
function ListItem(props) {
  const value = props.value;
  return (
    // Wrong! There's no need to specify the key here:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) => 
    // Wrong! The key should've been specified here:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

```
// Correct key usage
function ListItem(props) {
  // Correct! There is no need to specify the key here:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Correct! Key should be specified inside the array.
    <ListItem key={number.toString()} value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

- A good rul of thumb is that elements inside the `map()` call need keys

## Keys must only be unique among siblings

- Keys used within arrays should be unique among their siblings, but they **don't need to be globally unique**. One can use the same keys when producing two different arrays

```
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```

- Keys serve as a hint to React but they don't get passed to the components. If one need the same value in their component, pass it explicitly as a prop with a different name

```
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title}
  />
);
```

- From above, `Post` component can read `props.id` but not `props.key`

## Embedding map() in JSX

```
// Declaring separate variable to include in JSX
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```

- JSX also allows embedding any expression in curly braces so one could inline the `map()` result

```
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {number.map((number) =>
        <ListItem key={number.toString()} value={number} />
      )}
    </ul>
  );
}
```

- Keep in mind if the `map()` body is too nested then it might be a good time to extract a component