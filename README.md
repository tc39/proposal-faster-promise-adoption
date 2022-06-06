# proposal-faster-promise-adoption

The Faster Promise Adoption proposal aims to reduce the number of ticks
required for an outer promise to adopt the state of an inner promise.

Champions: [@jridgewell][jridgewell], (and you?)

Author: [@jridgewell][jridgewell]

Status: [Stage 1](https://tc39.es/process-document/)

## Problem

```javascript
const outer = new Promise(res => {
  const inner = Promise.resolve(1);
  res(inner);
});
```

In the current specification, it requires 2 ticks for the outer promise
to finally "settle" with the inner promise's `1` value. This comes up in
initial promise resolution (like above), chained promises `p.then(() =>
Promise.resolve(1))`, and with async functions:

```javascript
// Directly returning a promise settles `direct` after 2 ticks.
const direct = (async () => Promise.resolve(1))();

// Returning an awaited promise's value settles `awaited` after 1 ticks.
const awaited = (async () => await Promise.resolve(1))();
```

Surprisingly, it's actually faster to `return await promise` than to
`return promise`!

## Proposal

We'd like to make "fast path" for promise adoption that allows native
promises to quickly adopt the state of another native promise, without
opening up new reentrancy hazards with untrusted thenables. If possible,
we'd also like to native adoption of userland promises faster, but this
will only be done if we can determine that it's web compatible.

[jridgewell]: https://github.com/jridgewell
