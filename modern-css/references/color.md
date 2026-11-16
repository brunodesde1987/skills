# Color

Derive colors from one token instead of hard-coding every shade, and let
the browser pick readable text and theme-specific assets.

---

## Relative color syntax (Baseline newly available, 2024)

Take an origin color, read its channels, and change only what you need.

```css
/* ❌ OLD: a hand-picked hex per state, or Sass darken()/rgba() */
--brand: #4f46e5;
--brand-hover: #4338ca;
--brand-50: rgba(79, 70, 229, 0.5);

/* ✅ NEW: one token, derived states */
--brand: oklch(0.55 0.22 275);

.btn { background: var(--brand); }
.btn:hover { background: oklch(from var(--brand) calc(l - 0.08) c h); }
.btn:disabled { background: oklch(from var(--brand) l calc(c * 0.3) h); }
.overlay { background: oklch(from var(--brand) l c h / 50%); }
```

**Rules:**
- Every channel slot needs a value: write `l c h` to keep them, and
  `calc()` only on the channel you change.
- Use `oklch` as the output space so lightness steps look even across hues.

**Support:** Chrome 125+, Firefox 128+, Safari 18+.

---

## `alpha()` (Baseline newly available, 2026)

Shorthand for changing only the opacity of a color.

```css
/* ✅ NEW */
.overlay { background: alpha(from var(--brand) / 50%); }

/* Equivalent with relative color syntax, for older targets */
.overlay { background: oklch(from var(--brand) l c h / 50%); }
```

**Support:** Chrome 152+, Firefox 155+, Safari 27+. It is very new: prefer
the relative color form unless the targets are current.

---

## `color-mix()` (Baseline widely available, 2023)

Mix two colors in a chosen space.

```css
/* ❌ OLD: a separate token per tint */
/* ✅ NEW */
.badge { background: color-mix(in oklch, var(--brand) 20%, white); }
.divider { border-color: color-mix(in oklch, currentColor 25%, transparent); }
```

Mixing with `transparent` is the way to add opacity to a color whose
channels you can't read, such as `currentColor`.

**Support:** Chrome 111+, Firefox 113+, Safari 16.2+.

---

## `contrast-color()` (Baseline newly available, 2026)

Returns black or white, whichever contrasts more with the given color.

```css
/* ❌ OLD: JS luminance check, or a --text-on-brand token per brand color */
/* ✅ NEW */
.tag {
  background: var(--tag-color);
  color: contrast-color(var(--tag-color));
}
```

**When to use:** User-picked or data-driven background colors (labels,
avatars, charts). For designed palettes, explicit tokens give more control.

**Support:** Chrome 147+, Firefox 146+, Safari 26+.

---

## Gradient interpolation space (Baseline newly available, 2024)

```css
/* ❌ OLD: sRGB interpolation, muddy gray in the middle */
background: linear-gradient(blue, yellow);

/* ✅ NEW */
background: linear-gradient(in oklch, blue, yellow);

/* Rainbow the long way around the hue wheel */
background: linear-gradient(in oklch longer hue, red, red);
```

**Support:** Chrome 111+, Firefox 127+, Safari 16.2+.

---

## `light-dark()` with images (Baseline newly available, 2026)

`light-dark()` picks its first value in a light color scheme and its
second in a dark one. It needs `color-scheme: light dark` on the element
or an ancestor. Colors have worked since 2024; images and gradients are
newer.

```css
/* ❌ OLD: duplicate rules in a prefers-color-scheme media query */
.hero { background-image: url(hero-light.webp); }
@media (prefers-color-scheme: dark) {
  .hero { background-image: url(hero-dark.webp); }
}

/* ✅ NEW */
:root { color-scheme: light dark; }
.hero { background-image: light-dark(url(hero-light.webp), url(hero-dark.webp)); }
```

**Support:** colors: Chrome 123+, Firefox 120+, Safari 17.5+. Images:
Chrome 150+, Firefox 150+, Safari 27+.
