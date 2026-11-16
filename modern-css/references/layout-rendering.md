# Layout and Rendering

Centering, rendering cost, scrollbars, container conditions and text
features that used to need wrappers or scripts.

---

## `align-content` in block layout (Baseline widely available, 2024)

```css
/* ❌ OLD: flex or grid turned on only to center one child vertically */
.hero { display: flex; flex-direction: column; justify-content: center; }

/* ✅ NEW: block container, no display change */
.hero { min-block-size: 60vh; align-content: center; }
```

No wrapper and no `display` change, so the children keep block layout.

**Support:** Chrome 123+, Firefox 125+, Safari 17.4+.

---

## `content-visibility: auto` (Baseline newly available, 2025)

Skips layout and paint for off-screen sections until they approach the
viewport.

```css
/* ❌ OLD: JS virtualization or lazy-render wrappers for long static pages */
/* ✅ NEW */
.article-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 600px;
}
```

**Rules:**
- Always pair it with `contain-intrinsic-size: auto <estimate>` so the
  scrollbar doesn't jump; `auto` remembers the real size once rendered.
- Content stays in the DOM, so find-in-page still reaches it.
- It is for long documents and feeds of sections, not a replacement for
  virtualizing tens of thousands of rows.

**Support:** Chrome 108+, Firefox 130+, Safari 26+.

---

## Scrollbars: `scrollbar-gutter`, `scrollbar-width`, `scrollbar-color` (Baseline newly available, 2024–2025)

```css
/* ❌ OLD: overflow-y: scroll to avoid the shift, ::-webkit-scrollbar styling */
/* ✅ NEW */
.panel {
  overflow: auto;
  scrollbar-gutter: stable;          /* reserve space, no shift when it appears */
  scrollbar-width: thin;             /* auto | thin | none */
  scrollbar-color: var(--thumb) transparent; /* thumb, then track */
}
```

`scrollbar-width: none` hides the scrollbar but keeps scrolling. Only use
it when another affordance shows that the area scrolls.

**Support:** `scrollbar-gutter` Chrome 94+, Firefox 97+, Safari 18.2+.
`scrollbar-width` Chrome 121+, Firefox 64+, Safari 18.2+.
`scrollbar-color` Chrome 121+, Firefox 64+, Safari 26.2+.

---

## Container style queries (Baseline newly available, 2026)

Query a custom property on the container instead of adding a modifier
class to every child.

```css
/* ❌ OLD: .card--compact .title, .card--compact .meta, … */
/* ✅ NEW */
.card { --density: compact; }

@container style(--density: compact) {
  .title { font-size: 1rem; }
  .meta { display: none; }
}
```

Style queries need no `container-type`; every element is a style
container.

**Support:** Chrome 111+, Firefox 151+, Safari 18+.

---

## Name-only container queries (Baseline newly available, 2026)

Match on being inside a named container, with no size condition.

```css
.sidebar { container-name: sidebar; }

@container sidebar {
  .card { flex-direction: column; }
}
```

**Support:** Chrome 148+, Firefox 149+, Safari 26.4+.

---

## `text-indent: hanging` and `each-line` (Baseline newly available, 2026)

```css
/* ❌ OLD: negative text-indent plus padding for hanging indents */
.reference { padding-inline-start: 2em; text-indent: -2em; }

/* ✅ NEW: indent every line but the first */
.reference { text-indent: 2em hanging; }

/* Indent after forced line breaks too, not only the first line */
.poem { text-indent: 1em each-line; }
```

**Support:** Chrome 146+, Firefox 121+, Safari 15+.

---

## Custom highlights: `::highlight()` (Baseline newly available, 2026)

Style arbitrary text ranges (search hits, spell-check, syntax tokens)
without wrapping them in `<mark>` or `<span>` elements.

```js
// ❌ OLD: split text nodes and wrap each match in <mark>
// ✅ NEW: register ranges, no DOM changes
const ranges = findMatches(article, query); // returns Range objects
CSS.highlights.set('search', new Highlight(...ranges));
```

```css
::highlight(search) {
  background-color: var(--highlight);
  color: inherit;
}
```

Only a few properties apply (colors, text decoration, text shadow), and
the highlighted text is not exposed to assistive technology as marked.

**Support:** Chrome 105+, Firefox 149+, Safari 17.2+.
