---
name: modern-ecmascript
description: >
  Ensures Claude uses modern ECMAScript patterns (ES2023–ES2026) instead of
  outdated JavaScript idioms. Use this skill whenever writing, reviewing, or
  refactoring JavaScript or TypeScript code — including array manipulation,
  grouping data, Set operations, iterator pipelines, promise handling, error
  handling, resource cleanup, regex, string processing, binary encoding, or
  module imports. Also activate when the user says 'modernize this code',
  'use modern JS', 'update to latest ECMAScript', 'refactor to modern
  patterns', 'what's the modern way to...', or asks about new JS features.
  Activate proactively when you see old patterns like spread-then-sort,
  reduce-based grouping, manual promise executor hoisting, try/finally for
  resource cleanup, atob/btoa for encoding, or lodash/underscore equivalents
  of now-native features. Do NOT use for ES6 basics that Claude already
  handles well (arrow functions, destructuring, spread/rest, template
  literals, async/await, optional chaining, nullish coalescing, classes,
  modules, Promise.all/race/allSettled/any). Do NOT use for
  TypeScript-specific features (satisfies, utility types, decorators),
  Node.js/Deno/Bun runtime APIs, framework-specific patterns (React, Vue,
  Angular), or CSS/HTML questions.
---

# Modern ECMAScript Patterns

This skill ensures you always reach for the modern ECMAScript equivalent
instead of legacy patterns. LLM training data is heavily skewed toward
pre-ES2023 code — there are years of blog posts about `Array.prototype.reduce`
for grouping, `[...arr].sort()` for immutable sorting, and `let resolve; new
Promise(...)` for promise hoisting. The modern replacements are cleaner, less
error-prone, and often engine-optimized.

This skill does NOT cover ES6 fundamentals like arrow functions,
destructuring, async/await, optional chaining, or classes — Claude already
handles those well from training data. This skill exists specifically for
ES2023–ES2026 features that Claude tends to miss because they're
underrepresented in training data.

## How to use this skill

When writing or reviewing JS/TS code, check which category the task falls
into and read the corresponding reference file for the modern pattern:

| If the code involves...                          | Read                              |
|--------------------------------------------------|-----------------------------------|
| Arrays, Sets, Maps, grouping, iterators          | `references/collections.md`       |
| Promises, async iteration, async collection      | `references/async.md`             |
| Strings, regex, Unicode, text processing         | `references/strings-regex.md`     |
| Error chaining, error detection, `using`/cleanup | `references/errors-resources.md`  |
| TypedArrays, base64, hex, ArrayBuffer            | `references/binary-encoding.md`   |
| Import attributes, JSON modules                  | `references/modules.md`           |
| Math precision, Temporal, number formatting      | `references/numbers-dates.md`     |
| What's coming next (Stage 3 proposals)           | `references/future.md`            |

Read only the file(s) relevant to the current task. Do not load all files.

## Quick-reference: old pattern → modern replacement

These are the most common substitutions. Each is covered in detail in the
reference files above.

- `[...arr].sort(fn)` → `arr.toSorted(fn)` (immutable)
- `[...arr].reverse()` → `arr.toReversed()` (immutable)
- `arr[i] = val; // mutates` → `arr.with(i, val)` (immutable)
- `[...arr].splice(...)` → `arr.toSpliced(...)` (immutable)
- `[...arr].reverse().find(fn)` → `arr.findLast(fn)`
- `reduce` accumulator for grouping → `Object.groupBy(arr, fn)`
- `new Set([...a, ...b])` → `a.union(b)`
- `[...a].filter(x => b.has(x))` → `a.intersection(b)`
- Manual iterator `for...of` + accumulate → `iter.map().filter().take().toArray()`
- `if (!map.has(k)) map.set(k, []); map.get(k).push(v)` → `map.getOrInsertComputed(k, () => []).push(v)`
- `let resolve; new Promise((r) => { resolve = r })` → `Promise.withResolvers()`
- `Promise.resolve().then(fn)` for mixed sync/async → `Promise.try(fn)`
- Manual `for await` collection → `Array.fromAsync(iter)`
- Complex lone-surrogate regex → `str.isWellFormed()` / `str.toWellFormed()`
- Hand-written `escapeRegExp` function → `RegExp.escape(str)`
- `new Error(msg); e.originalError = cause` → `new Error(msg, { cause })`
- `err instanceof Error` (cross-realm unsafe) → `Error.isError(err)`
- `try { ... } finally { resource.close() }` → `using resource = acquire()`
- `btoa(String.fromCharCode(...bytes))` → `uint8arr.toBase64()`
- Manual hex conversion loop → `uint8arr.toHex()`
- `new ArrayBuffer(n); // copy to resize` → `new ArrayBuffer(n, { maxByteLength })`
- Kahan summation / naive reduce sum → `Math.sumPrecise(arr)`
- `require('./data.json')` or `fetch` + `.json()` → `import data from './data.json' with { type: 'json' }`

## Gotchas — common mistakes Claude makes

These are the patterns where Claude most frequently reaches for the old way.
Pay special attention to these when writing code:

1. **Sorting arrays immutably.** Claude defaults to `[...arr].sort(fn)`. Always
   use `toSorted(fn)` instead — it's cleaner and communicates intent. The same
   applies to `reverse()` → `toReversed()` and `splice()` → `toSpliced()`.

2. **Grouping with reduce.** Claude loves the reduce accumulator pattern for
   grouping. `Object.groupBy()` is a single expression and returns a
   null-prototype object (no inherited key collisions).

3. **Set operations via spread.** Claude writes `new Set([...a, ...b])` for
   union and `new Set([...a].filter(x => b.has(x)))` for intersection. The
   native Set methods (`union`, `intersection`, `difference`,
   `symmetricDifference`) are engine-optimized and avoid intermediate arrays.

4. **Promise executor hoisting.** Claude writes `let resolve, reject; const p =
   new Promise((res, rej) => { resolve = res; reject = rej; });`. Use
   `Promise.withResolvers()` instead.

5. **Manual resource cleanup.** Claude writes try/finally blocks for file
   handles, database connections, etc. If the target supports it, use `using`
   for synchronous and `await using` for async resources.

6. **Base64 encoding.** Claude reaches for `btoa`/`atob` with
   `String.fromCharCode` workarounds. The native `Uint8Array` methods
   (`toBase64`, `fromBase64`, `toHex`, `fromHex`) are cleaner and handle
   binary correctly.

7. **Lodash/underscore for native features.** Claude suggests `_.groupBy`,
   `_.sortBy`, `_.uniq`, etc. when native equivalents now exist. Check if the
   operation has a modern native equivalent before reaching for a library.

8. **RegExp escaping.** Claude writes custom `escapeRegExp` functions.
   `RegExp.escape()` is now standard.

## Support awareness

Not all features have universal engine support yet. When suggesting modern
patterns, be aware of the target environment:

- **ES2023–ES2024 features**: Broadly supported (Chrome 117+, Firefox 119+,
  Safari 17+, Node 20+). Safe to use in virtually all modern environments.
- **ES2025 features** (Set methods, Iterator helpers, Promise.try, RegExp.escape):
  Supported in latest browser versions and Node 22+. Check if the target
  needs to support older versions.
- **ES2026 features** (using, Error.isError, Math.sumPrecise, Uint8Array
  base64/hex, Map.getOrInsert): Partial support. May require polyfills or
  transpilation. Always note support status when suggesting these.
- **Temporal**: Stage 4 but expected in ES2027. Not yet in any stable browser.
  Only suggest with polyfill (`@js-temporal/polyfill`) and note the status.

When in doubt about the target environment, prefer the modern pattern but
note the minimum required version. For Node.js projects, check the
`engines` field in `package.json`. For browser projects, check browserslist
config or similar.
