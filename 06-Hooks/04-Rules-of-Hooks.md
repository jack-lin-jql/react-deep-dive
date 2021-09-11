# Rules of hooks

- Hooks are JS functions but one needs to follow 2 rules when using them

## Only call hooks at the top level

- Don't call hooks inside loops, conditions, or nested functions
- Always use hooks at the top level of React functions, before any early returns
- This ensures the **hooks are called in the same order each time a component renders**
  - That's what allows React to correctly preserve the state of hooks between multiple `useState` and `useEffect` calls

## Only call hooks from React functions

- Don't call hooks from regular JS functions
- Call hooks from React function components
- Call hooks from custom hooks
- This rule ensures the all stateful logic in a component is clearly visible from its source code

## Explanation

- Multiple states or effect hooks in a single component

```js
function Form() {
  const [name, setName] = useState('Mary');

  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  const [surname, setSurname] = useState('Poppins');

  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });
  
  // ...
}
```

- How does React know which state corresponds to which `useState` call?
- React relies on the order in which hooks are called!
- The example works because the order of the hook calls is the same on every render

```js
// ------------
// First render
// ------------
useState('Mary')           // 1. Initialize the name state variable with 'Mary'
useEffect(persistForm)     // 2. Add an effect for persisting the form
useState('Poppins')        // 3. Initialize the surname state variable with 'Poppins'
useEffect(updateTitle)     // 4. Add an effect for updating the title

// -------------
// Second render
// -------------
useState('Mary')           // 1. Read the name state variable (argument is ignored)
useEffect(persistForm)     // 2. Replace the effect for persisting the form
useState('Poppins')        // 3. Read the surname state variable (argument is ignored)
useEffect(updateTitle)     // 4. Replace the effect for updating the title

// ...
```

- As long as the order of the hook calls is the same between renders, React an associate some local state with each of them
- What happens if put inside a condition?

```js
// This breaks first rule
if (name !== '') {
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });
}
```

- Now, on first render the condition evals to true and the hook is run but the subsequent render might eval to false, so the order of the hook calls become different

```js
useState('Mary')           // 1. Read the name state variable (argument is ignored)
// useEffect(persistForm)  // 🔴 This Hook was skipped!
useState('Poppins')        // 🔴 2 (but was 3). Fail to read the surname state variable
useEffect(updateTitle)     // 🔴 3 (but was 4). Fail to replace the effect
```

- React wouldn't know what to return for the 2nd `useState` hook call
- React expects that the second hook call in this component corresponds to the `persistForm` effect like the previous render, but it doesn't, so every hook call after the one skipped would also be shifted by one, leading to bugs
- This is why hooks must be called on the top level of components
- Running an effect conditionally can simply encapsulate the condition within the effect function

```js
useEffect(function persistForm() {
  // Not breaking the first rule anymore
  if (name !== '') {
    localStorage.setItem('formData', name);
  }
});
```