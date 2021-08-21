# SyntheticEvent

- This documents the SyntheticEvent wrapped that forms part of React's Event System

## Overview

- One's event handlers will be passed instances of `SyntheticEvent`, a cross-browser wrapper around the browser's native event
- It has the same interface as the browser's native event, including `stopPropagation()` and `preventDefault()`, except the events work identically across all browsers
- If one finds that one needs the underlying browser event for some reason, simply use the `nativeEvent` attribute to get it
- The synthetic events are different from, and do not map directly to, the browser's native events
- E.g. in `onMouseLeave event.nativeEvent` will point to a `mouseout` event
- The specific mapping isn't part of the public API and may change at any time
- Every SyntheticEvent object has the following attributes:

```
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget
boolean defaultPrevented
number eventPhase
boolean isTrusted
DOMEvent nativeEvent
void preventDefault()
boolean isDefaultPrevented()
void stopPropagation()
boolean isPropagationStopped()
void persist()
DOMEventTarget target
number timeStamp
string type
```

- Note: As of v17, e.persist() doesn't do anything since the SyntheticEvent is no longer pooled
- Note: as by v0.14, returning false from an event handler will no longer stop event propagation. Instead, e.stopPropagation() or e.preventDefault() should be triggered manually, as appropriate

## Supported events

- React normalizes events so that they have consistent properties across different browsers
- The events handlers below are triggered by an event in the bubbling phase
- To register an event handler for the capture phase, append `Capture` to the event name; for example, instead of using `onClick`, one would use `onClickCapture` to handle the click event in the capture phase
  - Clipboard Events
  - Compositions Events
  - Keyboard Events
  - Focus Events
  - Form Events
  - Generic Events
  - Mouse Events
  - Pointer Events
  - Selection Events
  - Touch Events
  - UI Events
  - Wheel Events
  - Media Events
  - Image Events
  - Animation Events
  - Transition Events
  - Other Events

- Refer to the actual events [here](https://reactjs.org/docs/events.html#reference)