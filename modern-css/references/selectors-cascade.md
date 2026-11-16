# Selectors and Cascade

Native nesting, cascade control and selectors that replace preprocessor
features, naming conventions and specificity hacks.

---

## Nesting (Baseline widely available, 2023)

```css
/* ❌ OLD: preprocessor, or repeated parent selectors */
.card { padding: 1rem; }
.card:hover { background: var(--hover); }
.card .title { font-weight: 600; }

/* ✅ NEW */
.card {
  padding: 1rem;

  &:hover { background: var(--hover); }
  .title { font-weight: 600; }

  @media (width >= 600px) { padding: 2rem; }
}
```

**Rules:**
- `&` is required for compound selectors (`&:hover`, `&.active`) and is
  optional for descendants (`.title`).
- No string concatenation: `&-title` does not become `.card-title`.
- A nested selector list gets the specificity of its most specific
  member, as with `:is()`.

**Support:** Chrome 120+, Firefox 117+, Safari 17.2+.

---

## Cascade layers: `@layer` (Baseline widely available, 2022)

Order whole groups of styles so specificity only matters inside a group.

```css
/* ❌ OLD: !important, doubled classes, or #id to beat a library */
.btn.btn { color: var(--brand) !important; }

/* ✅ NEW: declare the order once, first layer loses */
@layer reset, vendor, base, components, utilities;

@import url('library.css') layer(vendor);

@layer components {
  .btn { color: var(--brand); }
}
```

**Rules:**
- Declare the order once, at the top of the entry stylesheet.
- Styles outside any layer beat every layer.
- `!important` reverses layer order: an important declaration in `reset`
  beats one in `utilities`.

**Support:** Chrome 99+, Firefox 97+, Safari 15.4+.

---

## `@scope` (Baseline newly available, 2026)

Limit styles to a subtree, with an optional lower boundary, and let the
closest scope win when scopes nest.

```css
/* ❌ OLD: BEM prefixes to stop styles leaking into nested components */
.card__image { border-radius: 8px; }

/* ✅ NEW: images in .card, but not inside its .content slot */
@scope (.card) to (.content) {
  img { border-radius: 8px; }
  :scope { padding: 1rem; } /* the .card itself */
}
```

Inside a `<style>` element, `@scope { … }` without a selector scopes to
the element's parent.

**Rules:**
- When two scoped rules have equal specificity, the one with the closer
  scope root wins (scope proximity), not the later one.
- Selectors inside `@scope` do not gain the scope root's specificity, unlike nesting.

**Support:** Chrome 143+, Firefox 146+, Safari 26.4+.

---

## `:nth-child(An+B of S)` (Baseline widely available, 2023)

Count only the siblings that match a selector.

```css
/* ❌ OLD: zebra stripes break when rows are hidden */
tr:nth-child(even) { background: var(--stripe); }

/* ✅ NEW: count visible rows only */
tr:nth-child(even of :not([hidden])) { background: var(--stripe); }

/* First three featured items */
.item:nth-child(-n + 3 of .featured) { outline: 2px solid; }
```

**Support:** Chrome 111+, Firefox 113+, Safari 9+.

---

## `:open` (Baseline newly available, 2026)

Matches elements in an open state, such as `<details>`, `<dialog>` and
`<select>`.

```css
/* ❌ OLD: attribute selectors, which miss pickers */
details[open] > summary { font-weight: 600; }

/* ✅ NEW */
details:open > summary { font-weight: 600; }
```

**Support:** Chrome 133+, Firefox 136+, Safari 26.5+.

---

## `:state()` for custom elements (Baseline newly available, 2024)

Style a custom element by its internal state, without reflecting it to an
attribute or class.

```js
class ToggleButton extends HTMLElement {
  #internals = this.attachInternals();
  set pressed(on) {
    on ? this.#internals.states.add('pressed') : this.#internals.states.delete('pressed');
  }
}
```

```css
/* ❌ OLD: toggle-button[data-pressed] or .is-pressed */
/* ✅ NEW */
toggle-button:state(pressed) { background: var(--active); }
```

**Support:** Chrome 125+, Firefox 126+, Safari 17.4+.

---

## `revert-rule` (Baseline newly available, 2026)

Resets a property to what it would be if the current rule were not there.
Its main use is the fallback branch of a conditional value, so a
condition can mean "leave it alone".

```css
.button {
  /* if() is limited availability, see future.md */
  background: if(style(--variant: ghost): transparent; else: revert-rule);
}
```

**Support:** Chrome 148+, Firefox 150+, Safari 27+.
