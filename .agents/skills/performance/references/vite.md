# Vite Performance Engineering Reference Manual

## 1. Development vs. Production Architecture Realities

Understanding Vite's dual architecture is critical to avoid false-positive performance diagnoses:

### Vite Development Mode (`vite` / dev server)
- **Unbundled Native ESM**: Vite serves source files as individual native ES modules over HTTP. An application with 300 components will trigger 300+ separate module network requests during initial dev page load.
- **HMR & Dev Wrappers**: Injects `/@vite/client`, `/@react-refresh`, and on-the-fly source transformations.
- **Pre-bundled Dependencies**: Dependencies in `/node_modules/.vite/` are converted to ESM via esbuild for fast startup.
- **Critical Warning**:
  - A development network trace showing hundreds of requests **does NOT mean** production will make excessive requests.
  - High development Total Blocking Time (TBT) **does NOT prove** high production TBT.
  - **Never use development request counts or dev TBT to conclude that production bundles are poorly optimized.**

### Vite Production Mode (`vite build` + `vite preview`)
- **Rollup Bundling**: Source files are tree-shaken, concatenated, chunked, and minified.
- **Production Baseline**: Always run `vite build` and test against `vite preview` to evaluate true production network payload and execution cost.

---

## 2. Production Chunking & Manual Chunks

Inspect `build.rollupOptions.output.manualChunks` in `vite.config.*`:
- **Vendor Splitting**: Isolating stable vendor libraries (e.g. React, UI libraries) into dedicated chunks allows browsers to cache them across app deployments.
- **Avoid Over-Splitting**: Creating 30+ tiny micro-chunks increases HTTP connection overhead and waterfall latency on high-latency mobile networks. Aim for balanced chunks (e.g. 50–200KB compressed).
- **Heavy Dependencies**: For massive libraries (Three.js, rich-text editors, complex chart engines), use dynamic imports at the usage site rather than static bundling.

---

## 3. Dynamic Imports & Code Splitting

```javascript
// Only load the heavy module when the feature is actually activated
const { init3DScene } = await import('./heavyScene.js');
```
- **The Invariant**: Never dynamically import above-the-fold hero markup or critical LCP elements.
- **Prime Candidates**: Modals, complex charts, below-the-fold interactive widgets, and route boundaries.

---

## 4. Asset Inlining Limits (`build.assetsInlineLimit`)

- By default, Vite inlines assets smaller than 4KB as base64 data URIs (`build.assetsInlineLimit: 4096`).
- **Hazard**: Setting `assetsInlineLimit` too high (e.g. 50KB) inlines large images into JavaScript chunks, inflating JS parse and execution time. Ensure large images and media remain separate static assets with proper caching headers.

---

## 5. CSS Code-Splitting

- Vite splits CSS along async chunk boundaries by default. Ensure critical above-the-fold styles are not inadvertently deferred behind dynamically imported chunks that execute after first paint.
