# Unofficial Preact FAQ

This is a list of common questions we frequently encounter in our [slack](https://preact-slack.now.sh/) channel.

## I'm seeing weird `$$typeof` properties on all objects with Preact X

This happens because the internal shape for `vnodes` (the data structure we use to compare components)
has changed with Preact X. It is not a class anymore. The `$$typeof`-property is set by `preact-compat`
which sets it on the prototype of a Preact 8 `vnode`. Uninstall it via `npm uninstall preact-compat` and
switch to `preact/compat' which is included in the `preact` npm package.

## How do you work with async data in Preact?

Call the async function via `useEffect` and store the result inside the component's `state`.
That way the component will automatically rerender when the async request is resolved.

```jsx
function Foo() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch("/my-api")
      .then(res => res.json())
      .then(res => setData(res));
  });

  return data===null ? "not loaded" : data;
}
```
