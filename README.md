# proposal-faster-promise-adoption

The Faster Promise Adoption proposal aims to reduce the number of ticks
required for an outer promise to adopt the state of an inner promise.

Champions: [@jridgewell][jridgewell], [@mhofman][mhofman], (and you?)

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

The origin of the problem was an attempt to [prevent synchronous interleaving
during thenable adoption][unwrapping-thenable] to guard the code performing the
adoption against re-entrancy hazards. Unfortunately other synchronous points of
interleaving remained or were introduced.

## Proposal

We'd like to make "fast path" for promise adoption that allows native
promises to quickly adopt the state of another native promise, while removing
all the re-entrancy hazards. If possible, we'd also like to make native adoption
of thenables (userland promises) faster, but this will only be done if we can
determine that it's web compatible, and does not allow for any synchronous
re-entrancy.

## Links

- June 2022 Committee Meeting: [slides][2022-06-slides], notes (pending publish).

[jridgewell]: https://github.com/jridgewell
[mhofman]: https://github.com/mhofman
[2022-06-slides]: https://docs.google.com/presentation/d/17QGvaa6G1XIc4LJj3ZvcjyVLwL6pXvG9Nk6S_eAgBiY/edit?usp=sharing
[unwrapping-thenable]: https://github.com/domenic/promises-unwrapping/issues/105
