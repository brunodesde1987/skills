# Future: Stage 3 Proposals

These proposals have complete spec text, reviewer sign-off, and test262
tests. They represent the near-future of JavaScript but are **not yet
finalized** — do not suggest them as if they're available today.

Use this file to stay aware of what's coming. If a user asks about
upcoming JS features, or if you're designing code that should be
future-compatible, reference this list.

---

## High likelihood of landing soon

### Decorators

`@decorator` syntax for classes and class elements. Enables declarative
metaprogramming — logging, validation, memoization, dependency injection.

```js
// What it will look like
@logged
class UserService {
  @validate
  async createUser(@required name, @email email) { ... }
}
```

**Replaces:** Manual wrapping functions, TypeScript experimental decorators
(which use a different, older spec). Note: the TC39 decorators spec differs
from TypeScript's legacy `experimentalDecorators` — they are not compatible.

### Source Phase Imports

`import source` for accessing module source without executing it.

```js
import source myModule from './worker.js';
const worker = new Worker(myModule);
```

**Replaces:** `new URL('./worker.js', import.meta.url)` pattern for Worker
instantiation and manual `fetch()` + module construction.

### Deferring Module Evaluation

`import defer` for lazy module loading — the module is fetched but not
executed until first property access.

```js
import defer * as utils from './heavy-utils.js';
// Module is fetched but NOT executed yet
// ...later...
utils.doThing(); // NOW it executes
```

**Replaces:** Dynamic `import()` with manual caching patterns for
performance-sensitive startup paths.

### Joint Iteration (Iterator.zip)

`Iterator.zipToArray()` and `Iterator.zipToObject()` for parallel
iteration over multiple iterables.

```js
// What it will look like
const keys = ['name', 'age', 'city'];
const values = ['Alice', 30, 'NYC'];

Iterator.zipToObject(keys, values);
// { name: 'Alice', age: 30, city: 'NYC' }

Iterator.zipToArray([iter1, iter2, iter3]);
// [[a1, b1, c1], [a2, b2, c2], ...]
```

**Replaces:** Manual zip functions, lodash `_.zip` / `_.zipObject`.

### Atomics.pause

CPU hint for spinlock efficiency — tells the processor to use minimal
resources while waiting.

**Replaces:** Busy-wait loops without CPU efficiency hints.

---

## Stage 2.7 (approaching Stage 3)

These are worth knowing about but should NOT be suggested in code:

- **ShadowRealm** — Isolated execution contexts for running untrusted code
  or creating clean global scopes.
- **Iterator Chunking** — `iterator.chunks(n)` for batching iterator elements
  into fixed-size arrays.
- **Await Dictionary** — `await` on an object's values, like `Promise.all`
  but with named keys instead of positional arrays.
- **Immutable ArrayBuffers** — `arrayBuffer.transferToImmutable()` for
  read-only shared memory.
- **Iterator Join** — `iterator.join(separator)` for joining iterator values
  into strings without `toArray()` first.

---

## How to use this information

1. **Don't suggest Stage 3 features as solutions.** They may change or
   stall. Focus on Stage 4 (finished) features in the other reference files.

2. **Design for compatibility.** If you're building something that a future
   proposal would improve, mention it in a code comment:
   `// TODO: Replace with Iterator.zip when available`

3. **Be cautious about TypeScript experimental decorators.** The TC39
   decorators spec (Stage 3) is different from TypeScript's legacy
   `experimentalDecorators`. Code using the old TS decorators will need
   migration when the standard lands.
