# Context-Aware Image Performance Reference Manual

## 1. Context-Aware Image Engineering

**File size alone is never a root cause.** Do not rely on blunt rules like *"image > 400KB = bad"*.
A 1.7MB uncompressed hero image can be directly catastrophic to Largest Contentful Paint (LCP). Conversely, a 1MB image placed far below the fold with native `loading="lazy"` may have virtually zero impact on initial load performance or Core Web Vitals.

Always evaluate the complete context of an image before recommending an optimization:
1. **Viewport Position**: Above the fold (critical path) vs. below the fold (deferred path).
2. **LCP Candidate Status**: Is this specific image the browser's designated LCP element?
3. **Intrinsic vs. Rendered Dimensions**: Is a 4000×3000px asset downloaded to fill a 400×300px card?
4. **Display Density (DPR)**: Does the image scale appropriately across 1x, 2x, and 3x screens?
5. **Format & Compression Efficiency**: Is modern AVIF or WebP used, or legacy uncompressed PNG?
6. **Request Timing & Waterfall**: When did the request start relative to navigation start?
7. **Cache & Delivery Policy**: Are immutable cache-control headers and edge CDN compression active?
8. **Visual Fidelity**: Will compression introduce visible artifacts into branding or graphics?

---

## 2. The LCP Image Protocol: Causal Optimization

Never assume the largest image file is automatically the LCP element.

Before suggesting preloading, priority changes, or re-encoding:
1. **Confirm the LCP Element**: Check the Lighthouse audit or DevTools Performance trace to confirm the exact DOM element driving LCP.
2. **Determine the Bottleneck Phase**:
   - **Resource Discovery Delay**: The image is buried in external CSS or injected late by client JavaScript → Add `fetchpriority="high"` to the `<img>` tag, or consider `<link rel="preload" as="image" href="..." fetchpriority="high">` in `<head>` only if discovered late.
   - **Resource Load Duration**: The file is oversized for the target viewport → Implement responsive `srcset`/`sizes` and modern format encoding (AVIF/WebP).
   - **Element Render Delay**: Image arrived, but main thread was blocked or layout was uncomputed → Fix render-blocking scripts or styles.

### Invariant Rules for Critical Images:
- **NEVER** put `loading="lazy"` on an above-the-fold or hero image. This forces the browser to wait until layout geometry is calculated before even initiating the network fetch, causing severe LCP delays.
- **NEVER** blindly preload every image. Preloading non-LCP images steals bandwidth from critical scripts and the actual LCP resource.
- **NEVER** lazy-load above-the-fold image components via `React.lazy()` merely to artificially reduce initial JavaScript bundle chunk sizes.

---

## 3. Layout Stability & Dimensions (CLS Prevention)

Layout shifts caused by unsized images damage Cumulative Layout Shift (CLS) and degrade user trust:

- **Explicit Intrinsic Dimensions**: Always provide integer `width` and `height` attributes on `<img>` tags representing the intrinsic aspect ratio:
  ```html
  <img
    src="/images/product-card.webp"
    width="800"
    height="600"
    alt="Product details"
    loading="lazy"
    decoding="async"
  >
  ```
- **CSS Aspect Ratio**: Pair with modern CSS to ensure fluid responsiveness without layout jumps:
  ```css
  img {
    max-width: 100%;
    height: auto;
    aspect-ratio: 16 / 9;
  }
  ```
- **Skeleton Placeholders**: When images are loaded dynamically via API, reserve the container bounding box using CSS aspect-ratio or skeleton placeholders.

---

## 4. Responsive Breakpoints & Modern Formats

- **Format Selection**: AVIF offers superior compression (50%+ smaller than JPEG at equal visual quality). WebP provides universal modern browser support (30%+ smaller). Offer AVIF with WebP fallback via `<picture>` where maximum savings are needed:
  ```html
  <picture>
    <source srcset="/images/hero-1200.avif 1200w, /images/hero-800.avif 800w" type="image/avif">
    <source srcset="/images/hero-1200.webp 1200w, /images/hero-800.webp 800w" type="image/webp">
    <img
      src="/images/hero-1200.jpg"
      width="1200"
      height="675"
      alt="Hero visual"
      fetchpriority="high"
      decoding="sync"
    >
  </picture>
  ```
- **Responsive `sizes` Attribute**: Always pair `srcset` with an accurate `sizes` attribute (e.g. `sizes="(max-width: 768px) 100vw, 50vw"`). Without `sizes`, browsers assume `100vw` across all viewports and download unnecessarily large files on desktop layouts.
- **Framework Image Components**: Utilize framework-native image components (`next/image`, `astro:assets`, etc.) when available, as they generate responsive `srcset` and reserve dimensions automatically.

---

## 5. Below-the-Fold Optimization Strategy

- Add `loading="lazy"` and `decoding="async"` to all images below the initial fold.
- For dense lists (e.g. 50+ images in an e-commerce catalog), pair native lazy loading with DOM virtualization so only visible rows are rendered into the DOM tree.
