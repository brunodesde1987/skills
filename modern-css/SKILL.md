---
name: modern-css
description: Modern CSS patterns from features that reached Baseline in 2023–2026, plus what is still limited. Replaces preprocessor nesting, specificity hacks, hand-mixed color variants, JS scroll listeners, nth-child index hacks and other outdated idioms with native CSS.
---

# Modern CSS Patterns

This skill makes you reach for the native CSS feature instead of the
workaround. Training data is full of Sass nesting, `!important` fights,
hard-coded hover shades, scroll listeners that toggle classes, and
`:nth-child(1)`…`:nth-child(10)` delay ladders. Browsers now ship native
replacements for all of them.

Status comes from the [web-features](https://github.com/web-platform-dx/web-features)
Baseline data:

- **Baseline widely available**: in Chrome, Edge, Firefox and Safari for
  30+ months. Use freely.
- **Baseline newly available (year)**: in all four, but recently. Use it,
  and check the project's browser targets for older versions.
- **Limited availability**: missing from at least one engine. Only with a
  fallback, and only when the user asks for it or the target allows it.

This skill does not cover CSS that models already write well (flexbox,
grid basics, custom properties, `clamp()`). If the `good-css` skill is
installed, it owns layout, interaction, motion and show/hide techniques;
use it for those and this skill for the features below.

## How to use this skill

Read only the file that matches what you are writing:

| If the code involves...                                        | Read                              |
|----------------------------------------------------------------|-----------------------------------|
| Nesting, specificity, `@layer`, `@scope`, selectors            | `references/selectors-cascade.md` |
| Color variants, translucency, readable text color, gradients   | `references/color.md`             |
| Rounding, stepping, staggering, ratios, typographic units, media queries | `references/math-units.md` |
| Centering, long pages, scrollbars, container queries, text indent, highlights | `references/layout-rendering.md` |
| Scroll-linked effects, easing curves, view transition groups, clip shapes | `references/animation.md` |
| Features not yet in every engine (`if()`, anchors, masonry…)   | `references/future.md`            |

## Quick-reference: old pattern → modern replacement

- Sass/Less nesting → native nesting with `&`
- `!important` or extra classes to win specificity → `@layer`
- BEM prefixes to stop styles leaking into children → `@scope (.card) to (.slot)`
- `:nth-child(2n)` then filtering by class in JS → `:nth-child(2n of .visible)`
- `details[open]`, `dialog[open]` → `:open`
- Hand-picked hover/active shades → `oklch(from var(--brand) calc(l - 0.1) c h)`
- Separate `--brand-50` alpha tokens → `oklch(from var(--brand) l c h / 50%)`
- JS luminance check for text on a dynamic background → `contrast-color(var(--bg))`
- Muddy gray midpoint in gradients → `linear-gradient(in oklch, …)`
- Two background images swapped by a dark-mode media query → `light-dark(url(…), url(…))`
- `calc()` + JS to snap to a grid → `round(up, 17px, 4px)`
- `:nth-child(1) { --i: 1 }` … ladders for stagger → `calc(sibling-index() * 50ms)`
- `@media (min-width: 600px) and (max-width: 799.98px)` → `@media (600px <= width < 800px)`
- Absolute positioning or flex wrapper just to center vertically → `align-content: center` on the block
- Layout shift when a scrollbar appears → `scrollbar-gutter: stable`
- JS lazy rendering of long lists of sections → `content-visibility: auto`
- `IntersectionObserver` / scroll listener toggling an animation class → `animation-timeline: view()`
- JS spring libraries for bounces `cubic-bezier()` can't express → `linear()` easing
- Same `::view-transition-group(name)` rule repeated per element → `view-transition-class`

## Gotchas: common mistakes

1. **Nesting is not Sass.** `&` is required for suffixes: `.card { &-title {} }`
   does not produce `.card-title`. Nested rules behave like `:is()`, so
   specificity is the highest of the selector list.

2. **Layer order is declared, not sourced.** Declare the order once at the
   top (`@layer reset, base, components, utilities;`). Unlayered styles
   beat every layer, and `!important` inside layers reverses the order.

3. **Relative color needs a channel for every slot.** `oklch(from x l c h)`
   reuses all three; write `calc()` only on the channel you change.

4. **`animation-timeline` goes after the `animation` shorthand.** The
   shorthand resets it to `auto`.

5. **Don't promote limited features to defaults.** `if()`, `@function`,
   anchor positioning, `corner-shape` and others in `references/future.md`
   are missing from at least one engine.

## Support awareness

Each entry ends with a **Support:** line from the web-features data. When
you suggest a Baseline newly available feature, name its year so the
user can compare it with their targets (browserslist config, or the
`engines` and support policy of the project). Limited-availability
features need a fallback that works without them.
