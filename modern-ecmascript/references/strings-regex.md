# String & Text Processing

Modern patterns for strings, regular expressions, and Unicode handling.

---

## Well-formed Unicode strings (ES2024)

`isWellFormed()` and `toWellFormed()` replace complex surrogate-pair regex
patterns and prevent `URIError` from `encodeURI` on lone surrogates.

```js
// ❌ OLD — fragile regex for lone surrogate detection
const hasLone = /(?:[^\uD800-\uDBFF]|^)[\uDC00-\uDFFF]|[\uD800-\uDBFF](?![\uDC00-\uDFFF])/.test(str);
const safe = str.replace(/[\uD800-\uDBFF](?![\uDC00-\uDFFF])|(?<![^\uD800-\uDBFF])[\uDC00-\uDFFF]/g, '\uFFFD');

// ✅ NEW — built-in, engine-optimized
str.isWellFormed();   // boolean
str.toWellFormed();   // replaces lone surrogates with U+FFFD (�)

// Practical: safe URL encoding
encodeURI(url.toWellFormed()); // never throws URIError
```

**When to use:** Any time you're passing user-supplied strings to APIs that
require well-formed UTF-16 (fetch, URL construction, TextEncoder,
WebSocket messages). Especially important when processing data from
external sources where encoding issues are common.

**Support:** Chrome 111+, Firefox 119+, Safari 16.4+, Node 20+.

---

## RegExp v flag — Unicode Sets (ES2024)

An upgrade to the `u` flag that adds **set intersection** (`&&`), **set
subtraction** (`--`), and **properties of strings** (multi-character matching).

```js
// ❌ OLD — verbose lookahead for "Greek letters only"
/(?=\p{Script_Extensions=Greek})\p{Letter}/u

// ✅ NEW — set intersection in character class
/[\p{Script_Extensions=Greek}&&\p{Letter}]/v

// Set subtraction: non-ASCII digits only
/[\p{Decimal_Number}--[0-9]]/v

// Multi-character property matching (emoji flag sequences)
/\p{RGI_Emoji_Flag_Sequence}/v.test('🇺🇳'); // true
```

**Key differences from `u` flag:**
- `v` and `u` are mutually exclusive — use one or the other.
- `v` treats `\p{...}` inside `[...]` as a set that supports `&&` and `--`.
- `v` enables `\p{...}` with "properties of strings" — patterns that can
  match sequences of multiple code points (like emoji flag sequences).

**Support:** Chrome 112+, Firefox 116+, Safari 17+, Node 20+.

---

## Duplicate named capture groups (ES2025)

Allows the same group name in different regex alternatives. Previously,
you had to use different names for the same logical field.

```js
// ❌ OLD — different names, fragile extraction
const re = /(?<year1>\d{4})-(?<month1>\d{2})|(?<month2>\d{2})-(?<year2>\d{4})/;
const year = match.groups.year1 ?? match.groups.year2; // manual fallback

// ✅ NEW — same name, works regardless of which alternative matched
const re = /(?<year>\d{4})-(?<month>\d{2})|(?<month>\d{2})-(?<year>\d{4})/;
match.groups.year;  // always the year, no fallback needed
```

**Rule:** The same name can appear in different alternatives (`|`), but not
in the same alternative. Only the groups from the matched alternative are
populated; unmatched groups are `undefined`.

**Support:** Chrome 125+, Firefox 129+, Safari 17.4+, Node 22+.

---

## RegExp.escape (ES2025)

Replaces hand-written escape functions that inevitably miss edge cases.

```js
// ❌ OLD — incomplete, doesn't cover future syntax characters
function escapeRegExp(str) {
  return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

// ✅ NEW — comprehensive, future-proof
const safe = RegExp.escape("developer.mozilla.org");
new RegExp(`https?://${RegExp.escape(userDomain)}(?=/)`);
```

**Why the old way is broken:** Custom escape functions can miss newly added
regex syntax characters. `RegExp.escape` is maintained by the spec and will
always be current.

**Support:** Chrome 136+, Firefox 134+, Safari 18.2+, Node 23+.

---

## RegExp pattern modifiers (ES2025)

Apply or disable `i`, `m`, `s` flags for only part of a pattern using
`(?ims-ims:subexpression)` syntax.

```js
// ❌ OLD — flag applies to entire pattern, can't mix
/^[a-z]+$/i  // everything case-insensitive

// ✅ NEW — selective flag application
/^(?i:[a-z])[a-z]+$/     // first char case-insensitive, rest sensitive
/^[a-z](?-i:[a-z]+)$/i   // disable case-insensitivity for part

// Practical: match a keyword case-insensitively but preserve case in value
/(?i:select)\s+(\w+)/    // SELECT, Select, select — but capture value as-is
```

**When to use:** Composing regex patterns from configuration or user input
where different parts need different flag behavior. Also useful when
storing patterns in config files where the pattern itself needs to carry
its flag preferences.

**Support:** Chrome 125+, Firefox 132+, Safari 17+.

---

## Hashbang grammar (ES2023)

Formalizes `#!` as a valid comment at the start of a JS file. This was
already supported by Node.js, but is now part of the ECMAScript spec.

```js
#!/usr/bin/env node
// This is now valid ECMAScript, not just a de-facto convention
console.log('Hello from a CLI script');
```

**Support:** Chrome 74+, Firefox 67+, Safari 14.1+, Node 12+.

---

## JSON.parse source text (ES2026)

The reviver function gains a `context` parameter with the original JSON
text for primitive values — solving precision loss with large numbers.

```js
// ❌ OLD — precision lost, no recovery
JSON.parse('{"id": 12345678901234567890}');
// id: 12345678901234568000 — WRONG (JS number precision exceeded)

// ✅ NEW — original source text preserved in reviver
JSON.parse('{"id": 12345678901234567890}', (key, value, context) => {
  if (key === 'id') return BigInt(context.source);
  return value;
});
// id: 12345678901234567890n — CORRECT
```

**When to use:** APIs returning large IDs (Snowflake IDs, database bigints),
high-precision financial data, or any JSON with numbers that exceed
`Number.MAX_SAFE_INTEGER`.

**Support:** Chrome 114+, Safari 18+; Firefox partial.
