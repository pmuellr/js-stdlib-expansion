# Configurable Collection Keying

## Champion(s)

Bradley Farias: @bmeck

## Use Case

In the scenarios where Maps and Sets of data are not intended to be keyed by referential identity, this proposal would allow developers to create collections that are keyed off other data by providing a function to declare the identity of an value.

Two API changes are proposed to the JS spec:

* `new Set([value, ...], {hash})`
* `new Map([[key, value], ...], {hash})`

These changes would add a new argument to both `Set` and `Map` constructors that would be given an Object with a property `hash`.

The `hash` property would be a function with the signature of `value => id`. The `id` result of the function can be any type of value; the `id` will use [`SameValueZero`](https://tc39.github.io/ecma262/#sec-samevaluezero) when comparing to existing entries in the collection. For `Set`s the `hash` function runs on the `value`s of the `Set`, but on `Map`s the `hash` function is run on the `key`s of the `Map`.

Importantly, the result of the `hash` will be store on a `Map` and not the original key. This means that using `map.keys()` will show the results of the `hash` only.

## Example

```js
const users = new Set([], {
  hash({email}) {
    return email;
  }
});
function register({email, password}) {
  let user = {email, password};
  if (users.has(user)) {
    throw new Error(`user already exists`);
  }
  users.add(user);
}
```

## Example Desugared

```js
function hash({email}) {
  return email;
}
const users = new Map([]);
function register({email, password}) {
  let user = {email, password};
  if (users.has(hash(user))) {
    throw new Error(`user already exists`);
  }
  users.set(hash(user), user);
}
```

## Data on Other Languages and Existing JS Libraries

1. Compare naming

| Environment | Notes |
| ---- | ---- |
| Java | `value.hashCode()` |
| C# | `value.GetHashCode()` |
| Ruby | `value.hash` |
| Python | `value.__hash__()` |
| Immutable.js | [`value.hashCode()`](https://facebook.github.io/immutable-js/docs/#/hash) |

2. Compare API design

All environments use a single value parameter and return a numeric type.

A divergence in allowing non-integer types was made here to allow for more complicated schemes where referential identity is also desirable.
