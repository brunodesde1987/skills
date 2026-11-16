# Future: Limited Availability

These features are missing from at least one engine. Suggest one only
when the user asks for it or the project's targets include it, and always
with a fallback that works without it. Most can be layered on as
progressive enhancement behind `@supports`.

```css
@supports (corner-shape: squircle) {
  .card { corner-shape: squircle; }
}
```

---

## Logic and reuse

### `if()`

Inline conditionals on style queries, media queries or `@supports`.

```css
.button {
  background: if(style(--variant: ghost): transparent; else: var(--brand));
}
```

**Support:** Chrome 137+; Firefox and Safari not yet.

### `@function`

Custom functions that return a value.

```css
@function --fluid(--min, --max) {
  result: clamp(var(--min), 2vw + 1rem, var(--max));
}
h1 { font-size: --fluid(2rem, 4rem); }
```

**Support:** Chrome 139+; Firefox and Safari not yet.

### Typed `attr()`

Read an attribute as a length, color, number and so on, in any property.

```css
.bar { inline-size: attr(data-width type(<length>), 0px); }
```

**Support:** Chrome 133+, Firefox 155+; Safari not yet.

### `@custom-media` and `random()`

Named media queries and random values. Not shipped in any engine yet.

---

## Positioning and form controls

### Anchor positioning

Tether an element to another without JS positioning libraries.

```css
.trigger { anchor-name: --menu-trigger; }
.menu {
  position: absolute;
  position-anchor: --menu-trigger;
  position-area: block-end span-inline-end;
  position-try-fallbacks: flip-block;
}
```

**Support:** Chrome 157+, Safari 27+; Firefox not yet.

### Customizable `<select>`

Full styling of the native select and its picker.

```css
select,
::picker(select) { appearance: base-select; }
```

**Support:** Chrome 135+, Safari 27+; Firefox not yet.

### Interest invokers

The `interestfor` HTML attribute shows a target (tooltip, preview card)
on hover, focus or long-press, without JS.

```html
<a href="/profile/ana" interestfor="ana-card">Ana</a>
<div id="ana-card" popover="hint">…</div>
```

**Support:** Chrome 149+; Firefox and Safari not yet.

---

## Layout

### Grid lanes (masonry)

```css
.gallery {
  display: grid-lanes;
  grid-template-columns: repeat(auto-fill, minmax(14rem, 1fr));
  gap: 1rem;
}
```

**Support:** Safari 27.2+; Chrome and Firefox not yet.

### Gap decorations

Rules between grid or flex tracks without borders on the items.

```css
.grid { display: grid; column-rule: 1px solid var(--line); row-rule: 1px solid var(--line); }
```

**Support:** Chrome 149+; Firefox and Safari not yet.

### `reading-flow`

Make keyboard and screen reader order follow the visual order of a
reordered flex or grid layout.

```css
.grid { display: grid; reading-flow: grid-rows; }
```

**Support:** Chrome 137+; Firefox and Safari not yet.

### `margin-trim`

Drop the margins of children that touch the container's edges.

```css
.stack { margin-trim: block; }
```

**Support:** Safari 16.4+; Chrome and Firefox not yet.

---

## Sizing and animation

### `interpolate-size` and `calc-size()`

Animate to and from `auto` and other intrinsic sizes.

```css
:root { interpolate-size: allow-keywords; }
.panel { block-size: 0; overflow: clip; transition: block-size 300ms; }
.panel.open { block-size: auto; }
```

**Support:** Chrome 129+; Firefox and Safari not yet.

### Scroll-state container queries

Style a sticky header once it is stuck, or a carousel by whether it can
scroll further.

```css
.header { position: sticky; inset-block-start: 0; container-type: scroll-state; }
@container scroll-state(stuck: top) {
  .header-inner { box-shadow: var(--shadow); }
}
```

**Support:** Chrome 144+; Firefox and Safari not yet.

### CSS carousels: `::scroll-marker`, `::scroll-button()`

Generated dots and previous/next buttons for scroll containers.

```css
.carousel { scroll-marker-group: after; }
.carousel > li::scroll-marker { content: ''; }
.carousel::scroll-button(right) { content: '→'; }
```

**Support:** Chrome 135+; Firefox and Safari not yet.

---

## Shape and text

### `corner-shape`

Squircles, bevels and notches on top of `border-radius`.

```css
.card { border-radius: 2rem; corner-shape: squircle; }
```

**Support:** Chrome 139+; Firefox and Safari not yet.

### `text-wrap: pretty`

Avoids short last lines in paragraphs. Safe to apply now: browsers that
lack it wrap as usual.

```css
p { text-wrap: pretty; }
```

**Support:** Chrome 117+, Safari 26+; Firefox not yet.

### Unprefixed `line-clamp`

Not shipped in any engine yet. Use the prefixed form, which only works
with `display: -webkit-box` and `-webkit-box-orient: vertical`:

```css
.excerpt {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
  overflow: hidden;
}
```
