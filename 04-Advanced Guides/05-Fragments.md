# Fragments

- A common pattern in React fro a component to return multiple elements
- Fragments lets one group a list of children without adding extra nodes to the DOM

```
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```

## Motivation

- A common pattern is for a component to return a list of children

```
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```

- `<Columns />` would need to return multiple `<td>` elements in order for the rendered HTML to be valid
- If parent div was used inside the `render()` of `<Columns />`, then the result HTML will be invalid

```
class Column extends React.Component {
  render() {
    return (
      <div>
        <td>Hello</td>
        <td>World</td>
      </div>
    );
  }
}
```

- Results in a `<Table />` output of:

```
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

- Fragments solve this problem

## Usage

```
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```

- Which results in a correct `<Table />` output of:

```
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```

## Short syntax

```
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```

- One can use `<></>` the same way one'd use any other element **except that it doesn't support keys or attributes**

## Keyed fragments

- Fragments declared with the explicit `<React.Fragment>` syntax may have keys
- A use case for this is mapping a collect to an array of fragments

```
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        //Without the `key`, React will fire a key warning
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description></dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```

- `key` is the only attribute that can be passed to `Fragment`
- Event handlers might be supported in the future