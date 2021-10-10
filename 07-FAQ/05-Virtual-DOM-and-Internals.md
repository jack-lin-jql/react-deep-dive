# Virtual DOM and internals

## What is the virtual DOM?

- A programming concept where an ideal or virtual representation of a UI is kept in memory and synced with the "real" DOM by a library such as ReactDOM. This process is called reconciliation
- This enables the declarative API of React: one tells React what state one wnats the UI to be in and it makes sure the DOM matches that state
- This abstracts out the attribute manipulation, event handling, and manual DOM updating that one would otherwise have to use to build an app
- In React world, "virtual DOM" is usually associated with React elements since they're objects representing the UI
- React, however, also uses internal objects called "fibers" to hold additional information about the component tree. They may also be considered a part of the VDOM implementation in React

## Is the shadow DOM the same as virtual DOM?

- No, shadow DOM is a browser technology designed primarily for scoping variables and CSS in web components
- The VDOM is a concept implemented by libraries in JS on top of browser APIs

## What is React fiber?

- The new reconciliation engine in React 16. Its main goal is to enable incremental rendering of the VDOM