# CSS Performance Engineering Reference Manual

## 1. Render-Blocking CSS & Discovery Waterfalls

A synchronous `<link rel="stylesheet">` blocks First Contentful Paint (FCP) and Largest Contentful Paint (LCP) until download and parse completion:
- **Eliminate `@import` Chains**: Never use `@import url(...)` inside CSS files. Each `@import` creates a sequential discovery waterfall: browser downloads CSS A → parses CSS A → discovers `@import` for CSS B → initiates download for CSS B. Replace `@import` rules with parallel `<link>` tags in HTML or bundle them at build time.
- **Critical CSS Inlining**: For static marketing pages, inlining critical above-the-fold CSS directly in `<head>` eliminates the stylesheet network round-trip.
- **Unused CSS Pruning**: Large design frameworks (unpurged Tailwind, full Bootstrap) ship hundreds of kilobytes of unused selectors. Ensure production build purging/content scanning is active.

---

## 2. Layout-Triggering vs. GPU-Composited Properties

The CSS properties targeted in animations and transitions dictate whether the browser recalculates layout geometry (Reflow) or delegates rendering directly to the GPU Compositor:

| Pipeline Stage | CSS Properties | Performance Impact |
| :--- | :--- | :--- |
| **Layout (Reflow)** | `top`, `left`, `right`, `bottom`, `width`, `height`, `margin`, `padding`, `border-width` | **Extremely Expensive**: Forces CPU to recompute geometry across parent, sibling, and child nodes. Guaranteed frame drops during continuous animation. |
| **Paint** | `color`, `background-color`, `border-color`, `box-shadow`, `border-radius` | **Expensive**: Forces CPU/GPU rasterization of affected pixel bitmaps. |
| **Composite** | `transform` (`translate3d`, `scale`, `rotate`), `opacity` | **Extremely Fast**: Handled on separate GPU compositor thread without triggering layout or paint. |

*Golden Rule*: Animate only `transform` and `opacity`. Never animate `top`, `left`, `width`, or `height`.

---

## 3. `will-change` Discipline

`will-change` hints to the browser to promote an element to its own GPU compositor layer:
- **Targeted Application**: Apply `will-change: transform` only to elements currently animating or about to animate.
- **Strict Anti-pattern**: **Never apply `will-change: all`** or set `will-change` permanently across hundreds of DOM nodes. Layer creation consumes significant GPU VRAM, leading to memory pressure and thermal throttling on mobile devices.
- **Cleanup**: Remove `will-change` once an animation completes.

---

## 4. Paint & Composite Overhead

- **`backdrop-filter` & Heavy Blur**: `backdrop-filter: blur(20px)` and deep `box-shadow` with large blur radii require expensive multi-pass GPU convolutions on every frame. During rapid scrolling on mobile, heavy backdrop filters cause severe compositor frame drops.
- Scope blur effects to static overlay elements, or disable them on low-power devices via media queries (`@media (max-width: 768px)`).
