# Animation and Shapes

Scroll-linked animation, richer easing and shapes without scripts or SVG.
Wrap motion in `@media (prefers-reduced-motion: no-preference)`.

---

## Scroll-driven animations (Baseline newly available, 2026)

Drive a keyframe animation by scroll position instead of time.

```css
/* ❌ OLD: IntersectionObserver or a scroll listener toggling classes */

/* ✅ NEW: reveal as the element enters the viewport */
@media (prefers-reduced-motion: no-preference) {
  .reveal {
    animation: fade-in linear both;
    animation-timeline: view();
    animation-range: entry 0% cover 30%;
  }
}

/* Reading progress bar tied to page scroll */
.progress {
  transform-origin: left;
  animation: grow linear both;
  animation-timeline: scroll(root block);
}

@keyframes fade-in { from { opacity: 0; translate: 0 2rem; } }
@keyframes grow { from { scale: 0 1; } }
```

**Rules:**
- Write `animation-timeline` after the `animation` shorthand, which
  resets it.
- `view()` tracks the element crossing its scroll port; `scroll()` tracks
  a scroller's position (`root`, `nearest` or `self`, then the axis).
- `animation-range` picks the slice of the timeline: `entry`, `exit`,
  `cover`, `contain`, each with a percentage.
- Use `linear` timing: scroll position already gives the pacing.

**Support:** Chrome 115+, Firefox 159+, Safari 26+.

---

## `linear()` easing (Baseline widely available, 2023)

A piecewise-linear curve through any number of points, which can
approximate springs and bounces that `cubic-bezier()` cannot.

```css
/* ❌ OLD: JS spring library for a simple overshoot */
/* ✅ NEW */
:root {
  --ease-bounce: linear(0, 0.36 9%, 1 18%, 0.81 27%, 1 37%, 0.95 47%, 1 58%, 1);
}
.toast { transition: translate 600ms var(--ease-bounce); }
```

Generate the points with a tool rather than by hand; more points give a
smoother curve.

**Support:** Chrome 113+, Firefox 112+, Safari 17.2+.

---

## `animation-composition` (Baseline widely available, 2023)

Combine an animation with the element's existing value instead of
replacing it.

```css
/* ❌ OLD: wrapper elements so a hover scale doesn't wipe a base rotate */
/* ✅ NEW */
.sticker { transform: rotate(-6deg); }
.sticker:hover {
  animation: pop 300ms;
  animation-composition: add; /* rotate(-6deg) then the animated scale */
}
@keyframes pop { 50% { transform: scale(1.1); } }
```

**Support:** Chrome 112+, Firefox 115+, Safari 16+.

---

## `view-transition-class` (Baseline newly available, 2025)

Give many view-transition elements one shared style. Each element still
needs its own `view-transition-name`; `match-element` generates one per
element (Chrome 137+, Firefox 144+, Safari 18.4+).

```css
/* ❌ OLD: the same rule for every name */
::view-transition-group(card-1),
::view-transition-group(card-2),
::view-transition-group(card-3) { animation-duration: 400ms; }

/* ✅ NEW */
.card {
  view-transition-name: match-element;
  view-transition-class: card;
}
::view-transition-group(*.card) { animation-duration: 400ms; }
```

**Support:** Chrome 125+, Firefox 144+, Safari 18.2+.

---

## `shape()` for `clip-path` (Baseline newly available, 2026)

CSS-syntax paths with responsive units, curves and arcs, instead of SVG
`path()` strings in fixed pixels.

```css
/* ❌ OLD: path() with absolute coordinates that can't flex */
clip-path: path('M0 0 H300 V180 Q150 220 0 180 Z');

/* ✅ NEW: percentages, so it scales with the element */
.banner {
  clip-path: shape(
    from 0% 0%,
    line to 100% 0%,
    line to 100% 85%,
    curve to 0% 85% with 50% 100%,
    close
  );
}
```

**Support:** Chrome 135+, Firefox 148+, Safari 18.4+.
