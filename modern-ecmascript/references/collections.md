# Collections & Iteration

Modern patterns for arrays, Sets, Maps, grouping, and iterator pipelines.

---

## Immutable array operations (ES2023)

The Change Array by Copy proposal added non-mutating counterparts to
existing mutating methods. These return new arrays, leaving the original
untouched — critical for React/Redux state management and functional patterns.

### toSorted / toReversed / toSpliced / with

```js
// ❌ OLD — spread-copy before mutation
const sorted = [...items].sort((a, b) => a - b);
const reversed = [...items].reverse();
const replaced = [...items]; replaced[2] = 99;
const spliced = [...items]; spliced.splice(1, 2, 10, 20);

// ✅ NEW — single expression, immutable, chainable
const sorted = items.toSorted((a, b) => a - b);
const reversed = items.toReversed();
const replaced = items.with(2, 99);
const spliced = items.toSpliced(1, 2, 10, 20);
// original array unchanged in all cases
```

**Why it matters:** The old spread-copy pattern creates a full copy, then
mutates it — which is wasteful and unclear in intent. The new methods
communicate immutability at the call site. They also work on frozen arrays
(where mutating methods throw) and are chainable:
`items.toSorted(fn).toReversed().with(0, 'first')`.

**Support:** Chrome 110+, Firefox 115+, Safari 16+, Node 20+.

### findLast / findLastIndex (ES2023)

```js
// ❌ OLD — reverse then find (wrong index, wasteful copy)
const last = [...items].reverse().find(x => x > 3);
// or manual loop
let last; for (let i = items.length - 1; i >= 0; i--) {
  if (items[i] > 3) { last = items[i]; break; }
}

// ✅ NEW — direct, correct index
const last = items.findLast(x => x > 3);
const lastIdx = items.findLastIndex(x => x > 3);
```

**Why:** Avoids creating a reversed copy. `findLastIndex` returns the correct
index in the original array (the reversed approach returns the wrong index).

**Support:** Chrome 97+, Firefox 104+, Safari 15.4+, Node 18+.

---

## Grouping (ES2024)

### Object.groupBy / Map.groupBy

```js
// ❌ OLD — reduce accumulator ceremony
const grouped = items.reduce((acc, item) => {
  const key = item.type;
  if (!acc[key]) acc[key] = [];
  acc[key].push(item);
  return acc;
}, {});

// ✅ NEW — one expression
const grouped = Object.groupBy(items, ({ type }) => type);
// { vegetables: [...], fruit: [...] }
```

**Key details:**
- `Object.groupBy` returns a **null-prototype object** — no inherited keys
  like `toString` or `hasOwnProperty` that could collide with your data.
- `Map.groupBy` returns a `Map`, allowing **non-string keys** (objects,
  symbols, etc.):

```js
const restock = { min: 5 };
const sufficient = { min: 0 };
const result = Map.groupBy(inventory, ({ quantity }) =>
  quantity < 6 ? restock : sufficient
);
result.get(restock); // items that need restocking
```

- These are **static methods**, not on Array.prototype — you call
  `Object.groupBy(arr, fn)`, not `arr.groupBy(fn)`.

**Support:** Chrome 117+, Firefox 119+, Safari 17.4+, Node 21+.

---

## Set methods (ES2025)

Seven native set algebra operations that replace manual filter-and-spread.

```js
const a = new Set([1, 2, 3]);
const b = new Set([3, 4, 5]);

// ❌ OLD — intermediate arrays, losing Set benefits
const union = new Set([...a, ...b]);
const inter = new Set([...a].filter(x => b.has(x)));
const diff = new Set([...a].filter(x => !b.has(x)));

// ✅ NEW — engine-optimized, no intermediate arrays
a.union(b);                // Set {1, 2, 3, 4, 5}
a.intersection(b);         // Set {3}
a.difference(b);           // Set {1, 2}
a.symmetricDifference(b);  // Set {1, 2, 4, 5}

// Boolean operations
a.isSubsetOf(b);           // false
a.isSupersetOf(b);         // false
a.isDisjointFrom(b);       // false (they share 3)
```

**Key detail:** All methods accept **set-like objects** (anything with `.size`,
`.has()`, `.keys()`), not just Set instances. This means you can pass a Map
(checks keys) or any custom object implementing the set-like protocol.

**Support:** Chrome 122+, Firefox 127+, Safari 17+, Node 22+.

---

## Iterator helpers (ES2025)

Lazy, chainable methods on `Iterator.prototype` that bring Array-like
ergonomics to generators, Map/Set iterators, and DOM collections.

### Lazy methods (return new Iterator)
`map`, `filter`, `take`, `drop`, `flatMap`

### Eager methods (return a value)
`reduce`, `toArray`, `forEach`, `some`, `every`, `find`

```js
// ❌ OLD — impossible to chain on iterators without converting to array
function* naturals() { let i = 0; while (true) yield i++; }

// Must manually break, can't compose
const result = [];
for (const n of naturals()) {
  if (n * n % 2 === 0) result.push(n * n);
  if (result.length >= 5) break;
}

// ✅ NEW — lazy pipeline, works on infinite sequences
const result = naturals()
  .map(n => n * n)
  .filter(n => n % 2 === 0)
  .take(5)
  .toArray();  // [0, 4, 16, 36, 64]
```

### Iterator.from — wrap any iterable

```js
// Wrap a non-iterator iterable (or legacy iterator-like) into a proper Iterator
const entries = Iterator.from(map.entries())
  .filter(([k, v]) => v > 10)
  .map(([k, v]) => k);
```

### Practical examples

```js
// Process Map entries lazily
const expensiveItems = new Map([['a', 100], ['b', 5], ['c', 200]])
  .entries()
  .filter(([, price]) => price > 50)
  .map(([name]) => name)
  .toArray(); // ['a', 'c']

// Paginated API — take only what you need
async function* fetchPages(url) { /* yields pages */ }
const firstThreeItems = fetchPages('/api/items')
  .flatMap(page => page.items)
  .take(3);

// DOM collections
document.querySelectorAll('a')
  .values()
  .map(el => el.href)
  .filter(href => href.startsWith('https'))
  .toArray();
```

**Support:** Chrome 122+, Firefox 131+, Safari 18.4+, Node 22+.

---

## Iterator.concat (ES2026)

Lazily chains multiple iterables without generator boilerplate.

```js
// ❌ OLD — generator wrapper
const combined = function* () { yield* a; yield* b; yield* c; }();

// ✅ NEW — clean, lazy
const combined = Iterator.concat(a, b, c);
// Composes with helpers
Iterator.concat(headers, rows, footers).filter(fn).take(10);
```

**Support:** Chrome 146+, Firefox 147+, Safari 26.4+.

---

## Map.getOrInsert (ES2026)

Atomizes the check-then-set pattern — reduces three hash lookups to one.

```js
// ❌ OLD — 3 lookups (has + get + set), verbose, race-prone
if (!map.has(key)) map.set(key, []);
map.get(key).push(value);

// ✅ NEW — single atomic operation
map.getOrInsertComputed(key, () => []).push(value);

// With a static default value (no factory function needed)
map.getOrInsert(key, 0);
```

**When to use which:**
- `getOrInsert(key, defaultValue)` — when the default is a primitive or a
  value you're fine creating even if the key exists.
- `getOrInsertComputed(key, () => value)` — when the default is expensive
  to create (arrays, objects, class instances) and you want lazy initialization.

**Support:** Chrome 145+, Firefox 144+, Safari 18.4+.

---

## Symbols as WeakMap keys (ES2023)

Non-registered Symbols can now serve as WeakMap/WeakSet/WeakRef keys.

```js
// ❌ OLD — empty objects as placeholder keys
const key = {}; // heavyweight, no semantic meaning
weakMap.set(key, metadata);

// ✅ NEW — lightweight, descriptive
const key = Symbol('component-metadata');
weakMap.set(key, metadata);
```

**Why:** Symbols are lighter than objects and can cross ShadowRealm boundaries.
Note: only non-registered symbols work — `Symbol.for('...')` symbols are not
garbage-collectable and cannot be WeakMap keys.

**Support:** Chrome 109+, Firefox 146+, Safari 16.4+, Node 20+.
