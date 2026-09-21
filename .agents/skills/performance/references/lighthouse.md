# Lighthouse & Core Web Vitals Engineering Reference

## 1. Core Engineering Principle: Symptoms vs. Root Causes

Automated audits like Lighthouse report **symptoms** and **heuristic opportunities**, not verified root causes.
- **Never follow**: `Lighthouse Recommendation → Automatic Code Modification`.
- **Always follow**: `Symptom (Lighthouse) → Evidence (DevTools/Traces) → Root-Cause Hypothesis → Experiment → Verification`.

A Lighthouse diagnostic saying *"Reduce unused JavaScript: 180KB"* does not prove that removing those bytes will improve your critical metric. The bytes might be in an asynchronously loaded chunk that never executes during initial paint, having zero effect on LCP or TBT. Always trace the actual execution path.

---

## 2. Environment Detection: Development vs. Production Realities

Always identify the runtime environment before drawing performance conclusions:

### Development Mode (Vite dev, Next.js dev, Webpack dev server)
- Characterized by Hot Module Replacement (HMR) sockets, unbundled ES module waterfalls (`/@vite/client`, `/@react-refresh`, `/node_modules/.vite/...`), and unminified source code.
- **Critical Warning**: Development Total Blocking Time (TBT) and CPU execution time are heavily distorted by dev-mode instrumentation.
  - A development network graph showing hundreds of module requests does **not** mean the production app makes excessive requests.
  - High development TBT does **not** prove production TBT is slow.
  - Use development mode for debugging code logic, but **never** use it as an authoritative performance baseline.

### Production Mode (`npm run build` + `npm run preview` / `next start`)
- Characterized by tree-shaken, minified, bundled chunks, production React runtime, and optimized asset pipelines.
- **Mandatory**: Establish all before-and-after baselines and validation measurements against the **Production Build**.

---

## 3. The Noise vs. Delta Rule (Repeated Measurements)

Single-run synthetic scores fluctuate due to background CPU tasks, thermal states, and network jitter:
- **Normal Measurement Noise**: Score fluctuations of ±3–5 points or millisecond-level shifts (e.g. LCP moving from `2.91s` to `2.86s`, or TBT moving from `210ms` to `195ms`) are within normal variance. **Never claim victory over measurement noise.**
- **Meaningful Evidence**: An LCP moving from `3.4s` to `1.8s`, or TBT dropping from `450ms` to `120ms` consistently across 2–3 runs represents genuine improvement.
- **Categorize Outcomes Rigorously**:
  - `IMPROVED`: Measurable delta exceeding noise threshold with zero regressions.
  - `LIKELY NOISE`: Small shift within standard variance band.
  - `REGRESSED`: Metrics worsened or functionality broke.
  - `INCONCLUSIVE`: High run-to-run variance; requires repeated runs or stricter CPU/network throttling.

---

## 4. Causal LCP Debugging: The 11-Factor Breakdown

Largest Contentful Paint (LCP) measures when the main visual content becomes visible. Deconstruct LCP into its 11 causal components:

1. **Exact Candidate Element**: Which DOM node is selected by the browser as the LCP element?
2. **Element Subsystem**: Is it an `<h1>` text block, an `<img>`, a CSS `background-image`, a `<video>` poster frame, a `<canvas>`, or an `<svg>`?
3. **Resource Discovery Time**: At what millisecond did the browser parser encounter the asset URL in the raw HTML?
4. **Request Dispatch Time**: Did the request fire immediately upon discovery, or was it queued behind blocking stylesheets or head scripts?
5. **Transfer Duration**: How long did network transmission take (bytes transferred vs. effective throughput)?
6. **Element Render Delay**: How many milliseconds elapsed between the arrival of the final asset byte and the actual screen paint?
7. **Viewport & Fold Alignment**: Is the element above the fold across mobile, tablet, and desktop viewports?
8. **Lazy Loading Misplacement**: Was `loading="lazy"` improperly placed on an above-the-fold hero element?
9. **Client-Side JS Dependency**: Did client-side JavaScript hydration delay the creation or rendering of the DOM node?
10. **Render-Blocking CSS**: Did synchronous `<link rel="stylesheet">` or `@import` rules block first paint?
11. **Bandwidth Competition**: Did non-critical assets (below-the-fold images, fonts, secondary scripts) saturate the connection ahead of the LCP asset?

*Rules of Engagement:*
- Prioritize the genuine LCP resource only when evidence proves it is the primary bottleneck.
- **Never blindly preload arbitrary assets.** Preloading non-critical assets steals network bandwidth from the true LCP element.
- **Never lazy-load critical hero content** merely to artificially shrink initial JavaScript chunk sizes.

---

## 5. Core Web Vitals Standards

Google's Core Web Vitals represent real-world user experience thresholds:

### Largest Contentful Paint (LCP)
- **Thresholds**: Good: ≤ 2.5s | Needs Improvement: 2.5s – 4.0s | Poor: > 4.0s
- **Primary Causes**: Slow server TTFB, render-blocking stylesheets/scripts, delayed asset discovery, uncompressed hero media, client-side rendering lag.

### Cumulative Layout Shift (CLS)
- **Thresholds**: Good: ≤ 0.10 | Needs Improvement: 0.10 – 0.25 | Poor: > 0.25
- **Primary Causes**: Media (`<img>`, `<video>`, `<iframe>`) without explicit `width`/`height` or CSS `aspect-ratio`, dynamically injected DOM content above existing elements, FOIT/FOUT font swaps without metric overrides, layout-triggering animations (`height`, `top`, `margin`).

### Total Blocking Time (TBT) & Interaction to Next Paint (INP)
- **Thresholds (TBT)**: Good: ≤ 200ms | Needs Improvement: 200ms – 600ms | Poor: > 600ms
- **Thresholds (INP)**: Good: ≤ 200ms | Needs Improvement: 200ms – 500ms | Poor: > 500ms
- **Primary Causes**: Long Tasks (>50ms) during page load or interaction, heavy hydration costs, un-debounced event listeners (scroll, resize), unoptimized third-party scripts.

### First Contentful Paint (FCP) & Speed Index (SI)
- **Thresholds (FCP)**: Good: ≤ 1.8s | Needs Improvement: 1.8s – 3.0s | Poor: > 3.0s
- **Measures**: Time until the first DOM content (text, image, non-white canvas) renders.
- **Primary Causes**: Render-blocking CSS `@import` chains, large synchronous `<script>` tags in `<head>`, slow server response.

---

## 6. Main-Thread Diagnostics: The 5 Questions for Long Tasks

For any Long Task (>50ms) identified in the Performance profiler, answer:
- **WHO**: Which script, library, or component initiated the execution?
- **WHAT**: What operation was executing (React component render, JSON deserialization, layout recalculation, shader compilation)?
- **WHEN**: Did the task fire during initial load, hydration, after user input, or on a repeating timer?
- **HOW LONG**: Exact execution duration in milliseconds?
- **WHY**: Why was this computation performed synchronously on the main thread rather than batched, deferred, or delegated to a worker?

---

## 7. Lab Data vs. Field Data Reality

- **Lighthouse (Lab Data)**: Synthetic, controlled environment with simulated CPU and network throttling. Excellent for deterministic debugging and before-and-after experiments.
- **Chrome User Experience Report (CrUX / Field Data)**: Real-world user telemetry across diverse physical devices, operating systems, and network conditions.
- When lab data and field data disagree: Use lab data to understand *mechanics* and debug bottlenecks; trust field data for actual *user impact*.
