# Math, Units and Media Queries

Math functions and units that remove JS measurement, `:nth-child` ladders
and off-by-one breakpoints.

---

## `round()`, `mod()`, `rem()` (Baseline newly available, 2024)

```css
/* ❌ OLD: JS to snap a size to a 4px grid */
/* ✅ NEW */
.panel { height: round(up, 37vh, 4px); }   /* nearest multiple of 4px, rounding up */

.dial { rotate: mod(var(--turns), 360deg); }
```

`round()` takes `nearest` (default), `up`, `down` or `to-zero`. `mod()`
takes the sign of the divisor, `rem()` the sign of the dividend, like `%`
in JS.

**Support:** Chrome 125+, Firefox 118+, Safari 17.2+.

---

## `abs()` and `sign()` (Baseline newly available, 2025)

```css
/* Distance from the active item, whichever side it is on */
.item { opacity: calc(1 - abs(var(--offset)) * 0.2); }
```

**Support:** Chrome 138+, Firefox 118+, Safari 15.4+.

---

## Trigonometric functions (Baseline widely available, 2023)

`sin()`, `cos()`, `tan()`, `asin()`, `acos()`, `atan()`, `atan2()`.

```css
/* Place items on a circle without precomputed offsets */
.orbit > * {
  --angle: calc(sibling-index() * 360deg / sibling-count());
  translate: calc(cos(var(--angle)) * 8rem) calc(sin(var(--angle)) * 8rem);
}
```

**Support:** Chrome 111+, Firefox 108+, Safari 15.4+. (The example also
uses `sibling-index()`, below.)

---

## `sibling-index()` and `sibling-count()` (Baseline newly available, 2026)

The element's 1-based position among its siblings, and how many siblings
there are.

```css
/* ❌ OLD: one rule per position, or --i set inline from JS */
.list > :nth-child(1) { --i: 1; }
.list > :nth-child(2) { --i: 2; }
/* … */

/* ✅ NEW: staggered entrance */
.list > * { transition-delay: calc(sibling-index() * 50ms); }

/* Equal slices with no wrapper math */
.tabs > * { inline-size: calc(100% / sibling-count()); }
```

**Support:** Chrome 138+, Firefox 154+, Safari 26.2+.

---

## `progress()` (Baseline newly available, 2026)

`progress(value, start, end)` returns a number from 0 to 1 for where
`value` sits between `start` and `end`, clamped.

```css
/* ❌ OLD: clamp() with a hand-derived slope and intercept */
/* ✅ NEW: fade the hero title as the viewport narrows */
.hero-title { opacity: progress(100vw, 375px, 1280px); }

/* Interpolate any pair of values */
.card {
  --t: progress(100cqi, 300px, 900px);
  padding: calc(1rem + var(--t) * 2rem);
}
```

**Support:** Chrome 138+, Firefox 155+, Safari 26+.

---

## Typographic units: `lh`, `rlh`, `cap`, `rcap` (Baseline 2023–2026)

| Unit   | Equals                                    |
|--------|-------------------------------------------|
| `lh`   | the element's computed line height        |
| `rlh`  | the root element's line height            |
| `cap`  | the height of capital letters             |
| `rcap` | `cap` of the root element                 |

```css
/* ❌ OLD: em values that drift from the line height */
p + p { margin-block-start: 1.5em; }
.icon { block-size: 0.7em; }

/* ✅ NEW: spacing on the line grid, icons sized to capitals */
p + p { margin-block-start: 1lh; }
.icon { block-size: 1cap; }
```

**Support:** `lh` Chrome 109+, Firefox 120+, Safari 16.4+. `rlh` Chrome
111+, Firefox 120+, Safari 16.4+. `cap` Chrome 118+, Firefox 97+, Safari
17.2+. `rcap` Chrome 118+, Firefox 147+, Safari 17.2+.

---

## Media query range syntax (Baseline widely available, 2023)

```css
/* ❌ OLD: min/max pairs and .98px gaps to avoid overlap */
@media (min-width: 600px) and (max-width: 799.98px) { … }

/* ✅ NEW */
@media (600px <= width < 800px) { … }
@media (width >= 1200px) { … }
```

Works the same in `@container (inline-size >= 30rem)`.

**Support:** Chrome 104+, Firefox 102+, Safari 16.4+.
