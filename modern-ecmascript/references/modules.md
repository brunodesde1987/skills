# Module System

Modern patterns for importing data and non-JS assets.

---

## Import attributes (ES2025)

The `with` keyword syntax enables type-safe, declarative imports of
non-JavaScript modules. It replaces both `require()` for JSON and
`fetch()` + `.json()` workarounds.

```js
// ❌ OLD — CommonJS only
const config = require('./config.json');

// ❌ OLD — verbose fetch workaround in ESM
const config = await (await fetch('./config.json')).json();

// ✅ NEW — declarative, secure, standard
import config from './config.json' with { type: 'json' };
import styles from './theme.css' with { type: 'css' };
```

### Dynamic import with attributes

```js
// ❌ OLD
const data = await fetch('./data.json').then(r => r.json());

// ✅ NEW
const { default: data } = await import('./data.json', {
  with: { type: 'json' }
});
```

### Security benefit

The `type` attribute provides a **MIME type check**. If a server is
compromised and sends JavaScript instead of JSON at the imported URL,
the import fails instead of executing malicious code. This prevents
MIME-type confusion attacks.

```js
// If the server sends JS instead of JSON at this URL:
import data from './api/config' with { type: 'json' };
// → ImportError — type mismatch, code is NOT executed
```

### Re-exporting

```js
export { default as config } from './config.json' with { type: 'json' };
```

### Historical note

The earlier `assert` keyword syntax was deprecated in favor of `with`.
If you see `import ... assert { type: 'json' }` in old code, update it.

**Support:** Chrome 123+, Firefox 138+, Safari 17.2+, Node 20.10+.
