# Binary Data & Encoding

Modern patterns for TypedArrays, base64/hex conversion, and buffer management.

---

## Float16Array (ES2025)

Native half-precision (16-bit) floating-point typed array. Provides 50%
memory savings over Float32Array — essential for WebGPU, ML inference,
and GPU data interop.

```js
// ❌ OLD — polyfill or manual bit manipulation
import { Float16Array } from '@petamoriken/float16';

// ✅ NEW — native, zero-dependency
const f16 = new Float16Array([1.0, 1.5, 2.0]);
f16.BYTES_PER_ELEMENT;      // 2 (vs 4 for Float32, 8 for Float64)
Math.f16round(1.337);        // 1.3369140625

// DataView support
const view = new DataView(buffer);
view.getFloat16(0);          // read half-precision float
view.setFloat16(0, 1.5);    // write half-precision float
```

**When to use:** ML model weight loading, WebGPU vertex/texture data,
audio processing, any scenario where precision loss is acceptable in
exchange for memory savings.

**Support:** Chrome 129+, Firefox 129+, Safari 18.2+, Node 24+.

---

## Uint8Array base64/hex methods (ES2026)

Native binary-to-string conversion methods that replace the clunky
`atob`/`btoa` workarounds and manual hex loops.

### Base64

```js
// ❌ OLD — awkward string intermediary, doesn't handle binary properly
const base64 = btoa(String.fromCharCode(...bytes));
const decoded = new Uint8Array(
  [...atob(base64)].map(c => c.charCodeAt(0))
);

// ✅ NEW — direct, clean, handles binary correctly
const arr = new Uint8Array([72, 101, 108, 108, 111]);
arr.toBase64();                          // "SGVsbG8="
arr.toBase64({ alphabet: "base64url" }); // URL-safe variant

Uint8Array.fromBase64("SGVsbG8=");       // Uint8Array [72, 101, 108, 108, 111]
```

### Hex

```js
// ❌ OLD — manual conversion loop
function toHex(uint8arr) {
  return Array.from(uint8arr, b => b.toString(16).padStart(2, '0')).join('');
}
function fromHex(hex) {
  return new Uint8Array(hex.match(/.{2}/g).map(b => parseInt(b, 16)));
}

// ✅ NEW — single method call
const arr = new Uint8Array([72, 101, 108, 108, 111]);
arr.toHex();                       // "48656c6c6f"
Uint8Array.fromHex("48656c6c6f");  // Uint8Array [72, 101, 108, 108, 111]
```

### Streaming with setFromBase64 / setFromHex

For large data, you can decode directly into an existing buffer:

```js
const target = new Uint8Array(1024);
const { read, written } = target.setFromBase64(largeBase64String);
```

**Support:** Firefox 133+, Safari 18.2+; Chrome not yet shipped.

---

## Resizable ArrayBuffer (ES2024)

Buffers that can grow in-place without copying. TypedArray views
automatically track the buffer's current size.

```js
// ❌ OLD — full copy to "grow" a buffer
const oldBuf = new ArrayBuffer(8);
const newBuf = new ArrayBuffer(16);
new Uint8Array(newBuf).set(new Uint8Array(oldBuf)); // copy everything

// ✅ NEW — in-place resize, views auto-update
const buf = new ArrayBuffer(8, { maxByteLength: 1024 });
const view = new Uint8Array(buf);
console.log(view.length); // 8

buf.resize(16);
console.log(view.length); // 16 — auto-updated!
console.log(buf.resizable); // true
console.log(buf.maxByteLength); // 1024
```

**Growable SharedArrayBuffer** works similarly for multi-threaded scenarios:
```js
const shared = new SharedArrayBuffer(8, { maxByteLength: 1024 });
shared.grow(16); // grow() instead of resize() — can only increase
```

**When to use:** Dynamic data accumulation (WebSocket message buffers,
file reading, network streams), WebAssembly memory interop, any scenario
where you'd otherwise allocate increasingly larger buffers and copy.

**Support:** Chrome 111+, Firefox 128+, Safari 16.4+, Node 20+.

---

## ArrayBuffer.transfer (ES2024)

Zero-copy ownership transfer — moves buffer contents to a new ArrayBuffer,
detaching the original. The `.detached` property replaces hacky detection.

```js
// ❌ OLD — full copy, source still usable (ownership unclear)
const newBuf = oldBuf.slice(0);

// ✅ NEW — zero-copy move, source detached
const newBuf = oldBuf.transfer();
console.log(oldBuf.detached);       // true (can't use anymore)
console.log(oldBuf.byteLength);     // 0

// Transfer with a new size
const larger = buf.transfer(1024);   // move + resize in one operation

// Transfer to fixed-size (remove resizability)
const fixed = resizableBuf.transferToFixedLength();
```

**When to use:** Passing buffers between workers or modules where you want
to enforce single-ownership semantics (similar to Rust's move semantics).

**Support:** Chrome 114+, Firefox 122+, Safari 17.4+, Node 22+.

---

## Symbols as WeakMap keys (ES2023)

Covered in `collections.md` — non-registered Symbols can serve as
WeakMap/WeakSet/WeakRef keys, providing lightweight alternatives to
empty object sentinels.
