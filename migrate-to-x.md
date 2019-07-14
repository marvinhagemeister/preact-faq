# How to migrate to Preact X from Preact 8

## !! THIS DOCUMENT IS A WORK IN PROGRESS !!

Preact X brings many new exciting features such as `Fragments`, `hooks` and
much improved compatibility with the react ecosystem. We tried to keep any
breaking changes to the minimum possible, but couldn't eliminate all of them
completely without comprimising on our Feature set. This document is intended
to guide you through upgrading an existing Preact 8.x application to Preact X
and is divided in 2 main sections

- [Upgrading dependencies](#Upgrading-dependencies)
- [Getting your code ready](#Getting-your-code-ready)

## Upgrading dependencies

_Note: Throgout this guide we'll be using the `npm` client and the commands should
be easily applicable to other package managers such as `yarn`._

Let's begin! First install Preact X:

```bash
npm install preact@next
```

Because compat has moved to core, there is no need for `preact-compat` anymore.
Remove it with:

```bash
npm remove preact-compat
```

### Updating preact-related libraries

To guarantee a stable ecosystem for our users (especially for our enterprise users)
we've put Preact X related libraries behind a `tag`.
If you're using `preact-render-to-string` we need to update it to the version
that works with X.

| Library                   | Preact 8.x | Preact X |
| ------------------------- | ---------- | -------- |
| `preact-render-to-string` | 4.x        | 5.x      |
| `preact-router`           | 2.x        | 3.x      |
| `preact-jsx-chai`         | 2.x        | 3.x      |

**All libraries for X can be installed via the `next` tag. For example: `npm install preact-router@next`**

### Setting up aliases

This is pretty much the same as for Preact 8.x with the notable exception that
you need to change one character: `preact-compat` -> `preact/compat`.

#### Aliasing in `preact-cli`

This is done automatically for you. Just the upcoming version that will be
tagged as stable once Preact X is marked as stable:

```bash
npm install preact-cli@rc
```

#### Aliasing in `webpack`

To alias any package in webpack you need to add the the `resolve.alias` section
to your config. Depending on the configuration you're using this section may
already be present, but missing the aliases for Preact.

```js
const config = {
  //...snip
  resolve: {
    alias: {
        'react': 'preact/compat',
        'react-dom/test-utils': 'preact/test-utils',
        'react-dom': 'preact/compat', // Must be below test-utils
    },
}
```

#### Aliasing in `parcel`

Parcel uses the standard `package.json` file to read configuration options under
an `alias` key.

```json
{
  "name": "my-package",
  "alias": {
    "react": "preact/compat",
    "react-dom/test-utils": "preact/test-utils",
    "react-dom": "preact/compat"
  },
  "dependencies": {},
  "devDependencies": {}
}
```

#### Aliasing in `jest`

Similar to bundlers, `jest` allows to rewrite module paths. The syntax is a bit
different, than in say webpack, because it's based on regex. Add this to your
jest configuration:

```js
{
  "moduleNameMapper": {
    "react": "preact/compat"
    "react-dom/test-utils": "preact/test-utils"
    "react-dom": "preact/compat"
  }
}
```

### Third party libraries

Due to the nature of the breaking changes, some existing libraries may cease to
work with X. Most of them have been updated already following our beta schedule
but you may encounter one where this is not the case.

#### `preact-redux`

`preact-redux` is one of such libraries that hasn't been updated yet. The good
news is that `preact/compat` is much more react-complient and works out of the
box with the react bindings called `react-redux`. Switching to it will resolve
the situation. Make sure that you've aliased `react` and `react-dom` to
`preact/compat` in your bundler.

1. Remove `preact-redux`
2. Install `react-redux`

#### `mobx-preact`

Due to our increased compatibility with the react-ecosystem this package isn't
needed anymore. Use `mobx-react` instead.

1. Remove `mobx-preact`
2. Install `mobx-react`

#### `styled-components`

Preact 8.x only worked up to `styled-components@3.x`. With Preact X this barrier
is no more and we work with the latest version of `styled-components`. Make sure
that you've [aliased react to preact](#setting-up-aliases) correctly.

## Getting your code ready

### Using named exports

To better support tree-shaking we don't ship with a `default` export in preact
core anymore. The advantage of this approach is that only the code you need
will be included in your bundle.

```js
// Preact 8.x
import Preact from "preact";

// Preact X
import * as preact from "preact";

// Preferred: Named exports (works in 8.x and Preact X)
import { h, Component } from "preact";
```

_Note: This change doesn't affect `preact/compat`. It still has both named and a default export to remain compatible with react._

### Remove the 3rd argument to `render`

The `render` function has changed and works now out of the box like you'd
expect it to. Repeated renders will correctly render into the container instead
of always appending to it.

```js
// Preact 8.x
render(<App />, container, container.firstChild);

// Preact X
render(<App />, container);
```

### `props.children` is not always an `array`

In Preact X we can't guarantee `props.children` to always be of type `array`
anymore. This change was necessary to resolve parsing ambiguities in regards
to `Fragments` and components that return an `array` of children. In most cases
you may not even notice it. Only in places where you'll use array methods
on `props.children` directly need to be wrapped with `toChildArray`. This
function will always return an array.

```js
// Preact 8.x
function Foo(props) {
  // `.length` is an array method. In Preact X when `props.children` is not an
  // array, this line will throw an exception
  const count = props.children.length;
  return <div>I have {count} children </div>;
}

// Preact X
import { toChildArray } from "preact";

function Foo(props) {
  const count = toChildArray(props.children).length;
  return <div>I have {count} children </div>;
}
```

### Don't access `this.state` synchronously

In Preact X the state of a component will no longer be mutated synchronously.
This means that reading from `this.state` right after a `setState` call will
return the previous values. Instead you should use a callback function to
modify state that depends on the previous values.

```jsx
this.state = { counter: 0 };

// Preact 8.x
this.setState({ counter: this.state.counter++ });

// Preact X
this.setState(prevState => {
  // Alternatively return `null` here to abort the state update
  return { counter: prevState.counter++ };
});
```

_Note: We're currently investigating if we can make this easier by shipping a `LegacyComponent` which has the old behaviour._