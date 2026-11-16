# Future: Stage 3 Proposals

These proposals have complete spec text, reviewer sign-off, and test262
tests. They represent the near-future of JavaScript but are **not yet
finalized** — do not suggest them as if they're available today.

Use this file to stay aware of what's coming. If a user asks about
upcoming JS features, or if you're designing code that should be
future-compatible, reference this list.

---

## Stage 3

### Await Dictionary (Promise.allKeyed)

`Promise.allKeyed` is `Promise.all` keyed by name instead of position,
the same shape as `Iterator.zipKeyed`.

```js
// What it will look like
const { shape, color, mass } = await Promise.allKeyed({
  shape: getShape(),
  color: getColor(),
  mass: getMass(),
});
```

**Replaces:** Positional `Promise.all` destructuring (easy to mismatch
order), and sequential `await`s that run requests one after another.

### Import Text

`type: 'text'` import attribute that imports a file's contents as a string.

```js
import template from './email.txt' with { type: 'text' };
```

**Replaces:** `fetch()` + `.text()` for module-relative assets. Deno and
Bun already support it.

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

### RegExp Buffer Boundaries

`\A` (start of input), `\z` (end of input) and `\Z` (end of input, before
an optional final line break) in Unicode-mode regexes. Unlike `^` and `$`,
they ignore the `m` flag.

```js
const re = /\Afoo|^bar/mu; // foo only at input start, bar at any line start
```

**Replaces:** Juggling `^`/`$` with and without the `m` flag, or
`(?-m:^)` modifier groups.

### Error Stack Accessor

Standardizes the existing `error.stack` property as an accessor on
`Error.prototype`. No new API: keep reading `err.stack` as today.

---

## Stage 2.7 (approaching Stage 3)

These are worth knowing about but should NOT be suggested in code:

- **Decorators** — `@decorator` syntax for classes and class elements, plus
  **Decorator Metadata**. Babel and TypeScript already ship transforms for
  it (see the TypeScript caveat below).

  ```js
  // What it will look like
  @logged
  class UserService {
    @validate
    async createUser(name, email) { ... }
  }
  ```
- **ShadowRealm** — Isolated execution contexts for running untrusted code
  or creating clean global scopes.
- **Import Bytes** — `import data from './file.bin' with { type: 'bytes' }`
  gives a `Uint8Array`.
- **JSON.parse options** — `JSON.parse(text, { freeze: true })` returns a
  deeply frozen result.
- **Immutable ArrayBuffers** — `arrayBuffer.transferToImmutable()` for
  read-only shared memory.

---

## How to use this information

1. **Don't suggest Stage 3 features as solutions.** They may change or
   stall. Focus on Stage 4 (finished) features in the other reference files.

2. **Design for compatibility.** If you're building something that a future
   proposal would improve, mention it in a code comment:
   `// TODO: Replace with Promise.allKeyed when available`

3. **Be cautious about TypeScript experimental decorators.** The TC39
   decorators spec (Stage 2.7) is different from TypeScript's legacy
   `experimentalDecorators`. Code using the old TS decorators will need
   migration when the standard lands.
