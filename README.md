# driver

Driver is a framework agnostic, zero dependency, tiny utility for organizing boolean logic trees.

After working with state machines, I realized I was tracking UI states via a plethora of boolean flags, often intermixing const/let variables with inline ternary logic. This is inevitable when working with a library like react.

I wanted a way to carefully craft explicit states and derive common values from them. For example, a particular button component may have several states, but will always need to know:

1. is the button disabled?
2. what is the button text?

and other common values like

3. what is the popover/warning text if the button is disabled?

By segmenting our UIs into explicit states, we can design and extend our UIs in a more pragmatic and extensible way. Logic is easier to reason about, organize, and test – and we can extend that logic without manipulating inline ternary expressions or fighting long lists of complex boolean logic.

## Installation

```bash
$ yarn add @switz/driver
```

or

```bash
$ npm i @switz/driver
```

## Simple example:

Every driver (this whole project needs a better name) reduces down to a single state. The first key in `states` to be true is the active state. Think of these as effectively if/else if statements.

```javascript
  const demoButton = driver({
    states: {
      isNotRecorded: !!match.config.dontRecord,
      isUploading: !match.demo_uploaded,
      isUploaded: !!match.demo_uploaded,
    },
    flags: {
      isDisabled: ['isNotRecorded', 'isUploading'],
      text: {
        isNotRecorded: 'Demo Disabled',
        isUploading: 'Demo Uploading...',
        isUploaded: 'Download Demo',
      },
    },
  });
```

The flags are pulled from the state keys. You can pass a function (and return any value), an array to mark boolean flags, or you can pass an object with the state keys, and whatever the current state key is will return that value.

`isDisabled` above returns a boolean value out of the states, whereas `text` returns a string that corresponds to the currently active state value.

Here's a simple example of how to use this in React/UI code:

```javascript
  <Button icon="download" disabled={!!demoButton.isDisabled}>
    {demoButton.text}
  </Button>
```

Now instead of tossing ternary statements and if else and tracking messy declarations, all of your ui state can be derived through a simpler and concise state-machine inspired function.

## Code this replaces

This is what I see a lot of UI code end up being:

```javascript
const DemoButton = ({ match }) => {
  const isUploadedText = match.demo_uploaded ? 'Demo Uploaded' : 'Demo Uploading...';

  return (
    <Button icon="download" disabled={match.config.dontRecord || !match.demo_uploaded}>
      {match.config.dontRecord ? 'Demo Not Recorded' : isUploadedText}
    </Button>
  );
}
```

Touching this code is a mess, keeping track of the state tree is hard, and interleaving state values, boolean logic, and so on is cumbersome. You could write this a million different ways.

#### Now this becomes

```javascript
const DemoButton = ({ match }) => {
  const demoButton = driver({
    states: {
      isNotRecorded: !!match.config.dontRecord,
      isUploading: !match.demo_uploaded,
      isUploaded: !!match.demo_uploaded,
    },
    flags: {
      isDisabled: (states) => states.isNotRecorded || states.isUploading,
      text: {
        isNotRecorded: 'Demo Disabled',
        isUploading: 'Demo Uploading...',
        isUploaded: 'Download Demo',
      },
    },
  });

  return (
    <Button icon="download" disabled={!!demoButton.isDisabled}>
      {demoButton.text}
    </Button>
  );
}
```

The code is "longer", but all of the logic has been moved outside of the actual JSX, it's easy to understand what is supposed to be happening at a glance, and it's far more organized.

The goal here is not to have _zero_ logic inside of your actual view, but to make it easier and more maintainable to design and build your view logic in some more complex situations.

## Docs

The `driver` function takes an object parameter with two keys: `states` and `flags`.

`states` is an object whose keys are the potential state values. Passing boolean values into these keys dictates which state key is currently active. The first boolean key is the active state.

`flags` is an object whose keys derive their values from what the current state key is. There are three interface for the `flags` object.

### States

```javascript
// states
{
  isNotRecorded: !!match.config.dontRecord,
  isUploading: !match.demo_uploaded,
  isUploaded: !!match.demo_uploaded,
},
```

### Flags

#### Function

You can return any value you'd like out of the function using the state keys

```javascript
// flags
{
  isDisabled: (states) => states.isNotRecorded || states.isUploading,
}
```

or you can access generated enums for more flexible logic

```javascript
// flags
{
  isDisabled: (state, stateEnums, activeEnum) => (activeEnum ?? 0) <= stateEnums.isUploading,
}
```

This declares that any state key _above_ isUploading means the button is disabled (in this case, `isNotRecorded` and `isUploading`).

#### Array

By using an array, you can specify a boolean if any item in the array matches the current state:

```javascript
// flags
{
  isDisabled: ['isNotRecorded', 'isUploading'],
}
```

This is the same as writing: `(states) => states.isNotRecorded || states.isUploading` in the function API above.

#### Object lookup

If you want to have a unique value per active state, an object map is the easiest way to declare as such. Each key maps to what the value is for the active state. For Example:

```javascript
// flags
{
  text: {
    isNotRecorded: 'Demo Disabled',
    isUploading: 'Demo Uploading...',
    isUploaded: 'Download Demo',
  },
}
```

If the current state is `isNotRecorded` then the `text` key will return `'Demo Disabled'`.

`isUploading` will return `'Demo Uploading...'`, and `isUploaded` will return `'Download Demo'`.

## This is naive

I cobbled this together in a few hours or so, it's not fully tested or complete. I do think I'll end up using it quite a bit, but I'll likely rewrite the API and rename it and change a lot. Please contribute if you have ideas, I'm open to everything.

## Typescript

The typing is weak at the moment. Suggestions here would be helpful on how to derive the flags' values dynamically based on their `states` keys and `flags` values.

Right now, you may have to specify the types of the values the `flags` return as they are marked as `unknown`. I'm not sure how to dynamically type these, though I think it is possible to do. If you know how, please let me know.

## Local Development

To install dependencies:

```bash
bun install
```

To test:

_waiting on bun to upgrade to TS 5 syntax support_

```bash
bun test
```
