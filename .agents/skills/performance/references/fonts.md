# Typography & Font Performance Reference Manual

## 1. Font Audit Checklist

When auditing typography and web fonts:
- **Formats in use**: Confirm whether modern WOFF2 is served or if legacy TTF/OTF/WOFF/EOT is being delivered.
- **File Sizes & Payloads**: Audit individual file weights (WOFF2 files should typically be <30KB each).
- **Variant Bloat**: Count the number of loaded font families, weights, and styles (e.g. 100, 200, 300, 400, 500, 600, 700, 800, 900 italic).
- **Critical vs. Non-Critical Fonts**: Identify which specific font is required for above-the-fold hero rendering versus secondary weights needed only in downstream sections.
- **Preload Discipline**: Verify which fonts are preloaded in `<head>`.

---

## 2. Preloading Rules: Preload Only Genuinely Critical Fonts

Preloading tells the browser to immediately allocate highest network priority to a resource.

- **Rule**: **Only preload the 1 or 2 genuinely critical font files** needed for above-the-fold body text and hero headlines:
  ```html
  <link rel="preload" href="/fonts/inter-bold.woff2" as="font" type="font/woff2" crossorigin>
  ```
- **Strict Anti-pattern**: **Never preload every font or variant.** Preloading 5+ font files starves the network connection, directly delaying critical hero images, stylesheets, and JavaScript bundles.
- Always include the `crossorigin` attribute when preloading fonts, even for self-hosted files, as the CSS font loading specification mandates anonymous CORS mode.

---

## 3. Format: Universal WOFF2 Standard

- WOFF2 provides 30–50% superior Brotli compression compared to WOFF, TTF, and OTF.
- WOFF2 is supported across all modern browsers (98%+ market share).
- Deliver WOFF2 as the primary self-hosted format. Maintain older formats only if strict legacy browser support is a documented project requirement.

---

## 4. Pruning Variant & Weight Bloat

- Each loaded font weight adds 20–60KB of network payload. Loading multiple unused weights (e.g. 300, 500, 600, 800, plus italics) can easily add 300KB+ of blocking font assets.
- Audit the actual CSS / styling declarations and eliminate font requests for weights not actively applied in the design system. Most production sites require only 2–3 variants (e.g. 400 Regular, 700 Bold).

---

## 5. `font-display` Strategy

- `@font-face` rules lacking `font-display` default to `block` in most browsers, concealing text for up to 3 seconds until the font arrives (Flash of Invisible Text / FOIT), which harms FCP and perceived speed.
- **`font-display: swap`**: Recommended for body copy and headlines where immediate legibility is prioritized. Displays the fallback system font instantly and swaps when the web font arrives.
- **`font-display: optional`**: Ideal for secondary or decorative typography where avoiding layout shift on slow connections is preferred over guaranteed custom font rendering.

---

## 6. Eliminating Layout Shift on Font Swap (CLS Prevention)

When a web font swaps in over a fallback font with different letter metrics, the entire layout can re-flow, causing severe Cumulative Layout Shift (CLS).

Use CSS metric-matched fallback overrides:
```css
@font-face {
  font-family: 'FallbackFont';
  src: local('Arial');
  size-adjust: 105%;
  ascent-override: 92%;
  descent-override: 24%;
  line-gap-override: 0%;
}

body {
  font-family: 'CustomFont', 'FallbackFont', sans-serif;
}
```
Framework features such as Next.js `next/font` automate metric override calculation to eliminate font swap CLS completely.
