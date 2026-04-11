# Error Handling & Resource Management

Modern patterns for error chaining, error detection, and deterministic
resource cleanup.

---

## Error.cause — standardized error chaining (ES2022)

Attaches the original error as a `cause` property via the options bag.

```js
// ❌ OLD — original error lost or ad-hoc attachment
try {
  await connectToDatabase();
} catch (err) {
  throw new Error("DB connection failed"); // cause gone!
  // or
  const wrapped = new Error("DB connection failed");
  wrapped.originalError = err; // non-standard, every lib does it differently
  throw wrapped;
}

// ✅ NEW — standardized, preserves full error chain
try {
  await connectToDatabase();
} catch (err) {
  throw new Error("DB connection failed", { cause: err });
}

// Accessing the chain
catch (e) {
  console.log(e.message);       // "DB connection failed"
  console.log(e.cause.message); // original DB error message
  console.log(e.cause.cause);   // if there's a deeper cause
}
```

**Why it matters:** Error chains are critical for debugging in production.
Without `cause`, you lose the original stack trace and error details when
wrapping errors at abstraction boundaries. The `cause` property works
with any Error subclass and is part of the standard error serialization.

**Support:** Chrome 93+, Firefox 91+, Safari 15+, Node 16.9+.

---

## Error.isError — reliable cross-realm detection (ES2026)

Uses an internal branded check instead of `instanceof`, which fails
across iframes, Workers, and vm contexts, and is easily spoofed.

```js
// ❌ OLD — unreliable across realms
error instanceof Error;                        // false for cross-iframe errors!
({ __proto__: Error.prototype }) instanceof Error; // true — false positive!

// ✅ NEW — branded check, works everywhere
Error.isError(new Error());                    // true
Error.isError(new TypeError());                // true (all native error types)
Error.isError(crossRealmError);                // true
Error.isError({ __proto__: Error.prototype }); // false (no spoofing)
Error.isError(new DOMException());             // true
```

**When to use:** Error handling in libraries that run across realms (iframes,
Workers, ShadowRealms), or any code that needs to reliably distinguish
errors from error-like objects.

**Support:** Chrome 136+, Firefox 134+; Safari not yet.

---

## Explicit Resource Management — `using` declarations (ES2026)

Brings RAII-style deterministic cleanup to JavaScript. Resources with a
`Symbol.dispose` or `Symbol.asyncDispose` method are automatically disposed
when the block scope exits — even on exception. Same concept as C# `using`,
Python `with`, and Rust's `Drop`.

### Synchronous: `using`

```js
// ❌ OLD — manual try/finally, easy to forget, error-swallowing
const handle = acquireFileHandle();
try {
  const data = handle.read();
  process(data);
} finally {
  handle.close(); // if this throws, the original error is lost
}

// ✅ NEW — automatic cleanup on scope exit
{
  using handle = acquireFileHandle();
  const data = handle.read();
  process(data);
} // handle[Symbol.dispose]() called automatically, even on throw
```

### Asynchronous: `await using`

```js
// ❌ OLD
const conn = await pool.getConnection();
try {
  await conn.query("SELECT * FROM users");
} finally {
  await conn.release(); // must remember to await
}

// ✅ NEW
{
  await using conn = await pool.getConnection();
  await conn.query("SELECT * FROM users");
} // conn[Symbol.asyncDispose]() called and awaited
```

### Making your objects disposable

```js
class DatabaseConnection {
  // Sync disposal
  [Symbol.dispose]() {
    this.close();
  }

  // Async disposal (for resources needing async cleanup)
  async [Symbol.asyncDispose]() {
    await this.flush();
    await this.close();
  }
}
```

### DisposableStack — managing multiple resources

```js
// ❌ OLD — deeply nested try/finally
const a = acquireA();
try {
  const b = acquireB();
  try {
    const c = acquireC();
    try {
      doWork(a, b, c);
    } finally { c.close(); }
  } finally { b.close(); }
} finally { a.close(); }

// ✅ NEW — flat, LIFO disposal order
{
  using stack = new DisposableStack();
  const a = stack.use(acquireA());
  const b = stack.use(acquireB());
  const c = stack.use(acquireC());
  doWork(a, b, c);
} // c disposed first, then b, then a (LIFO)

// Async equivalent
{
  await using stack = new AsyncDisposableStack();
  const conn = stack.use(await openConnection());
  const tx = stack.use(await conn.beginTransaction());
  await tx.execute(query);
} // tx disposed first, then conn
```

### SuppressedError — error safety during disposal

If the main code throws AND disposal also throws, both errors are preserved
in a `SuppressedError` instead of the disposal error silently swallowing
the original:

```js
try {
  using resource = getResource();
  throw new Error("main error");
  // resource[Symbol.dispose]() also throws "dispose error"
} catch (e) {
  e instanceof SuppressedError; // true
  e.error;      // the disposal error
  e.suppressed; // the original "main error"
}
```

### Practical patterns

```js
// File operations
async function processFile(path) {
  await using file = await openFile(path);
  const content = await file.readAll();
  return transform(content);
} // file closed automatically

// Database transactions with automatic rollback
async function transferFunds(from, to, amount) {
  await using tx = await db.beginTransaction();
  await tx.execute('UPDATE accounts SET balance = balance - ? WHERE id = ?', [amount, from]);
  await tx.execute('UPDATE accounts SET balance = balance + ? WHERE id = ?', [amount, to]);
  await tx.commit(); // if this succeeds, dispose is a no-op
  // if any line above throws, tx is rolled back via dispose
}

// Temporary files
{
  using tmpFile = createTempFile();
  await downloadTo(tmpFile.path);
  await processFile(tmpFile.path);
} // tmpFile deleted automatically

// Locks
{
  using lock = await acquireLock('resource-key');
  await criticalSection();
} // lock released
```

**Support:** Chrome 134+ (partial); Firefox and Safari not yet.
TypeScript 5.2+ has transpiler support. If your target doesn't support
`using` natively, you can still implement `Symbol.dispose` on your classes
for future compatibility while using try/finally in the meantime.
