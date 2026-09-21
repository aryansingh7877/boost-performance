# Next.js Performance Engineering Reference Manual

## 1. Server vs. Client Components Architecture (App Router)

The boundary between Server Components and Client Components directly impacts Total Blocking Time (TBT) and hydration overhead:
- **Default to Server Components**: Keep data fetching, markdown rendering, and static layout strictly on the server. Server Components ship 0 client-side JavaScript.
- **Push `'use client'` to the Leaves**: Apply `'use client'` to the smallest interactive element (e.g. an interactive button, modal trigger, or 3D canvas container) rather than the parent page or layout.
- **Root Layout Hazard**: Placing `'use client'` in `app/layout.tsx` or `app/page.tsx` forces the entire page hierarchy into client hydration, causing massive TBT spikes. Always keep root layouts as Server Components.

---

## 2. Dynamic Imports & The LCP Invariant

```tsx
import dynamic from 'next/dynamic';

// Defer non-critical heavy widgets behind hydration
const HeavyAnalyticsChart = dynamic(() => import('@/components/AnalyticsChart'), {
  ssr: false,
  loading: () => <div className="aspect-video bg-neutral-900 animate-pulse" />
});
```

### The Golden Rule:
> **NEVER use `dynamic(() => import(...), { ssr: false })` on critical above-the-fold hero elements or LCP candidates.**
>
> Doing so forces the browser to wait for client hydration before rendering the hero, severely degrading LCP. Reserve dynamic imports for below-the-fold components, modals, and interaction-gated widgets.

---

## 3. Image Optimization with `next/image`

- **LCP Hero Image**: Add the `priority` attribute to the genuine hero LCP image. This preloads the image and disables lazy loading.
- **Dimension Reservation**: Always specify `width` and `height`, or use `fill` with a styled parent container with explicit `aspect-ratio` to prevent Cumulative Layout Shift (CLS).
- **Responsive `sizes`**: When using `fill`, always provide an accurate `sizes` prop (e.g. `sizes="(max-width: 768px) 100vw, 50vw"`). Omitting `sizes` causes `next/image` to generate desktop-width images for mobile devices.

---

## 4. Typography with `next/font`

- Use `next/font/google` or `next/font/local`. It automatically self-hosts font files, inlines critical font declarations in `<head>`, and calculates metric-matched fallback overrides (`size-adjust`, `ascent-override`) to eliminate font swap layout shifts (CLS).
- Load only the specific weights actively used in design systems (e.g. `weights: ['400', '700']`).

---

## 5. Streaming & TTFB Optimization

- Use `loading.tsx` or `<Suspense>` boundaries around slow dynamic data-fetching subtrees so fast static layout paints immediately (protecting FCP and TTFB).
- Use Static Site Generation (SSG) or Incremental Static Regeneration (ISR via `revalidate`) for marketing, blog, and product pages to achieve near-instantaneous TTFB.
- **Middleware Hygiene**: Keep `middleware.ts` lightweight and avoid blocking database queries; slow middleware directly delays Time to First Byte across every matched route.
