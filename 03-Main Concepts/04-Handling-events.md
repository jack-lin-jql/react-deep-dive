# Handling events

- Event handling in React is very similar to handling events on DOM elements, only some syntax differences 
- React events are named using camelCase, rather than lowercase
- With JSX, one passes a function as the event handler instead of a string

```
// In plain HTML
<button onclick="activateLasers()">
  Activate Lasers
</button>
```

```
// In React
<button onClick={activateLasers}>
  Activate Lasers
</button>
```

- One cannot return `false` to present default behavior in React, one must call `preventDefault` explicitly

```
// In plain HTML
<form onsubmit="console.log("You clicked submit."); return false">
  <button type="submit">Submit</button>
</form>
```

```
// In React
function Form() {
  function handleSubmit(e) {
    e.preventDefault();
    console.log('You clicked submit.');
  }

  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Submit</button>
    </form>
  );
}
```