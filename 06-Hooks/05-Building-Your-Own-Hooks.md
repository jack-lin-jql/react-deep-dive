# Building your own hooks

- Lets one extract component logic into reusable functions

```js
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

- Now say that the chat app also has a contact list and we want to render names of online users with a green color
- One could copy and paste the similar logic into the `FriendListItem` component, but that's not ideal

```js
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

- We'd like to instead share the logic between `FriendStatus` and `FriendListItem`
- Traditionally, two popular ways of sharing stateful logic between components were render props and HOCs
- Now, hooks can solve the same problem without forcing one to add more components to the tree

## Extracting a custom hook

- When wanting to share logic between JS functions, one can extract it to a third function. Both components and hooks are functions, so this same concept applies
- **A custom hook is a JS function whose name starts with "use" and that may call other hooks**

```js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

- There is nothing new inside of the extracted custom hook, similar to a component, ensure to only call other hooks unconditionally a the top level of the custom hook
- Unlike React components, custom hooks don't need specific signature
- We decide what it takes as arguments and what if anything it should return
- It's just like a normal function, but its name should always start with `use` so one can tell at a glance that the rules of hooks apply to it

## Using a custom hook

```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }

  return isOnline ? 'Online' : 'Offline';
}
```

```js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return ( 
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

- If this code equivalent to the original examples? - Yes, they work in exactly the same way
  - The behavior are identical, all that was done was extraction of some common code between two functions into a separate function
  - **Custom hooks are a convention that naturally follows from the design of hooks, rather than a React feature**
- Do I have to name my custom hooks starting with "use"? - Please do. It's important and without it, React wouldn't be able to automatically check for violations of rules of hooks since it won't be able to tell if a certain function contains calls to hooks inside of it
- Do two components use the same hook share state? - No. Custom hooks are a mechanism to reuse stateful logic, but every time one uses a custom hook, all states and effects inside of them are full isolated
- How does a custom hook get isolated state? - Each call to a hook gets isolated state 
  - Since `useFriendStatus` was called directly, from React's PoV, the component just calls useState and useEffect
  - Calling useSTate and useEffect many times in one component will be completely independent

## Tip: Pass information between hooks

- Since hooks are functions, one can pass information between them 

```js
const friendList = [
  { id: 1, name: 'Phoebe' },
  { id: 2, name: 'Rachel' },
  { id: 3, name: 'Ross' },
];

function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    </>
  );
}
```

- The currently chosen friend ID is kept in a state variable and updates if the user chooses a different friend in the `<select>` picker
- Since `useState` hook call gives us the latest value of the `recipientID` state variable, it can be passed to the custom `useFriendStatus` hook as an argument

```js
const [recipientID, setRecipientID] = useState(1);
const isRecipientOnline = useFriendStatus(recipientID);
```

- This allows the indication of currently selected friend's online status
- When picking a different friend which updates the recipientID, the useFriendStatus hook will unsubscribe from the previously selected friend and subscribe to the status of the newly selected one