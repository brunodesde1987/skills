# Async Patterns

Modern patterns for promises, async iteration, and async collection.

---

## Promise.withResolvers (ES2024)

Extracts resolve/reject without nesting inside the executor callback.

```js
// ❌ OLD — variable hoisting, nested scope pollution
let resolve, reject;
const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});
// resolve and reject available outside

// ✅ NEW — clean destructuring, single expression
const { promise, resolve, reject } = Promise.withResolvers();
```

**When to use:** Any time resolve/reject need to be called from outside
the executor — event listeners, stream wrappers, deferred patterns,
custom async primitives, bridging callback-based APIs.

```js
// Practical: wrapping an event emitter
function waitForEvent(emitter, event) {
  const { promise, resolve, reject } = Promise.withResolvers();
  emitter.once(event, resolve);
  emitter.once('error', reject);
  return promise;
}

// Practical: creating a deferred queue
function createDeferred() {
  const { promise, resolve, reject } = Promise.withResolvers();
  return { promise, resolve, reject };
}
```

**Support:** Chrome 119+, Firefox 121+, Safari 17.4+, Node 22+.

---

## Promise.try (ES2025)

Wraps any callback — sync or async — into a Promise, catching synchronous
throws that `Promise.resolve(fn())` would miss.

```js
// ❌ OLD — two different failure modes
// Promise.resolve(fn()) — if fn() throws synchronously, it's uncaught
// Promise.resolve().then(fn) — defers execution (fn is called async)

// ✅ NEW — unified: calls fn synchronously, wraps result in Promise,
//          catches sync throws into rejection
Promise.try(fn)
  .then(handleResult)
  .catch(handleError); // catches BOTH sync throws and async rejections
```

**When to use:** Building libraries or APIs where a callback might be sync
or async and you want uniform error handling. Particularly useful at the
start of a promise chain where you don't know if the first function is
synchronous or asynchronous.

```js
// Practical: unified API handler
function processInput(input, transformer) {
  // transformer might be sync or async — Promise.try handles both
  return Promise.try(() => transformer(input))
    .then(validate)
    .then(save);
}
```

**Support:** Chrome 128+, Firefox 134+, Safari 18.2+, Node 23+.

---

## Array.fromAsync (ES2026)

Collects an async iterable into an array in one expression. Unlike
`Promise.all()`, it awaits values **sequentially** (respecting backpressure).

```js
// ❌ OLD — manual loop
const results = [];
for await (const item of asyncGenerator()) {
  results.push(item);
}

// ✅ NEW — single expression
const results = await Array.fromAsync(asyncGenerator());

// With a mapping function (like Array.from)
const doubled = await Array.fromAsync(asyncGenerator(), x => x * 2);

// Works with any async iterable
const fetched = await Array.fromAsync(
  urls.map(url => fetch(url).then(r => r.json()))
);
```

**Key difference from Promise.all:**
- `Promise.all` runs everything **concurrently** and fails fast.
- `Array.fromAsync` processes **sequentially**, awaiting each value before
  requesting the next. Better for rate-limited APIs or when ordering matters.

**Support:** Chrome 121+, Firefox 115+, Safari 16.4+, Node 22+.

---

## Atomics.waitAsync (ES2024)

Non-blocking version of `Atomics.wait()` that returns a Promise instead
of blocking the thread. Can be used on the **main thread** (unlike
`Atomics.wait` which is Workers-only).

```js
// ❌ OLD — blocks the thread, cannot run on main thread
const result = Atomics.wait(int32Array, 0, 0, 1000); // "ok" | "timed-out"

// ✅ NEW — non-blocking, works everywhere
const { async, value } = Atomics.waitAsync(int32Array, 0, 0, 1000);
if (async) {
  value.then(v => console.log(v)); // "ok" | "timed-out"
} else {
  console.log(value); // immediate result if already notified
}
```

**When to use:** Multi-threaded web applications, WebAssembly shared memory
coordination, and any shared-memory scenario where blocking is unacceptable.

**Support:** Chrome 90+, Firefox 145+, Safari 16.4+, Node 16+.

---

## Atomics.pause (ES2027)

A CPU hint for the spin phase of a lock: it waits a very short time and
lets sibling cores use shared resources. It does not block, so it works on
the main thread and in workers.

```js
// ❌ OLD — empty busy-wait loop, no CPU hint
while (!tryLock()) {}

// ✅ NEW — spin briefly with a hint, then fall back to a real wait
let spins = 0;
while (!tryLock()) {
  if (++spins < 100) Atomics.pause();
  else await Atomics.waitAsync(lockState, 0, LOCKED).value;
}
```

**When to use:** Lock and mutex implementations over `SharedArrayBuffer`,
and Emscripten-style compiled code. Application code rarely needs it.

**Support:** Chrome 133+, Firefox 137+, Safari 18.4+, Node 24+.
