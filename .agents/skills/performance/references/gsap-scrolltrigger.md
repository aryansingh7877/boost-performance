# Evidence-Based GSAP & ScrollTrigger Reference Manual

## 1. Prime Rule: Never Blame GSAP Without Direct Profiler Evidence

GSAP is an industry-standard, hyper-optimized animation engine. When scroll jank or high main-thread activity occurs, GSAP itself is rarely the root cause; improper architectural integration is almost always responsible.
- **Rule**: **Do NOT automatically blame GSAP or ScrollTrigger.**
- Only attribute a bottleneck to GSAP when direct evidence confirms it:
  - DevTools Performance trace identifies an expensive callback inside GSAP's tick handler causing Long Tasks (>50ms).
  - Forced synchronous reflow warnings originate from DOM geometry reads inside `onUpdate` or scroll callbacks.
  - Memory or listener accumulation is traced directly to uncleaned `ScrollTrigger` instances across route changes.
- **Preserve Choreography**: Retain exact animation timing, easing curves, spatial coordinates, and visual storytelling. Only optimize the underlying mechanics.

---

## 2. Component Lifecycle & Teardown (#1 Cause of GSAP Degradation)

In reactive component frameworks (React, Vue, Svelte), creating animations and `ScrollTrigger` instances without proper lifecycle teardown results in accumulation: orphaned triggers re-calculate bounds on every scroll tick indefinitely, degrading performance over time.

### React Teardown Pattern with `gsap.context()`:
```jsx
import { useEffect, useRef } from 'react';
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

export function AnimatedSection() {
  const containerRef = useRef(null);

  useEffect(() => {
    // gsap.context scopes all tweens and triggers created inside
    const ctx = gsap.context(() => {
      gsap.to('.animated-card', {
        y: -40,
        opacity: 1,
        scrollTrigger: {
          trigger: '.animated-card',
          start: 'top 80%',
          end: 'bottom 20%',
          scrub: true,
        },
      });
    }, containerRef);

    // ctx.revert() cleanly kills all animations and ScrollTriggers on unmount
    return () => ctx.revert();
  }, []);

  return <div ref={containerRef}>{/* content */}</div>;
}
```
*Note: In projects using `@gsap/react`, the `useGSAP()` hook encapsulates scoping and teardown automatically.*

---

## 3. Composited Properties Only: Zero Visual Change

Tweening layout properties (`top`, `left`, `width`, `height`) forces the browser into layout and repaint on every frame.
- **Rule**: Animate only GPU-composited transforms (`x`, `y`, `xPercent`, `yPercent`, `scale`, `rotation`) and `opacity`.
- Maintain exact visual motion while migrating property keys:
  ```javascript
  // ❌ Layout reflow on every scroll tick
  gsap.to('.box', { top: 100, width: 300 });

  // ✅ 60fps GPU compositor transform
  gsap.to('.box', { y: 100, scaleX: 1.5, transformOrigin: 'left center' });
  ```

---

## 4. Hot Callback Auditing & Layout Thrashing

- **Never Query DOM in Hot Loops**: Calling `document.querySelectorAll()` or `gsap.utils.toArray()` inside `onUpdate` or scroll listeners wastes CPU. Cache element references during initialization.
- **Eliminate Layout Reads in `onUpdate`**: Reading geometry (`getBoundingClientRect()`, `offsetTop`, `offsetHeight`) inside `onUpdate` while mutating styles causes forced synchronous reflows. Let ScrollTrigger manage scroll metrics internally.

---

## 5. Consolidating Dense Trigger Clusters

Having dozens of independently configured `ScrollTrigger` instances in a single viewport section adds computational overhead:
- **Timeline Staggering**: Group sequential element entrances into a single timeline governed by one master `ScrollTrigger`.
- **`ScrollTrigger.batch()`**: For large grids of items (e.g. 20+ cards), batch observer calculations:
  ```javascript
  ScrollTrigger.batch('.grid-item', {
    onEnter: (batch) => gsap.to(batch, { opacity: 1, y: 0, stagger: 0.08, overwrite: true }),
    onLeaveBack: (batch) => gsap.to(batch, { opacity: 0, y: 30, overwrite: true }),
  });
  ```

---

## 6. Lenis Smooth Scroll Synchronization

When combining Lenis smooth scrolling with GSAP ScrollTrigger:
- Synchronize Lenis's animation frame loop directly with GSAP's ticker to avoid competing `requestAnimationFrame` loops:
  ```javascript
  lenis.on('scroll', ScrollTrigger.update);
  gsap.ticker.add((time) => lenis.raf(time * 1000));
  gsap.ticker.lagSmoothing(0);
  ```

---

## 7. Responsive Choreography with `ScrollTrigger.matchMedia()`

Adapt animation complexity across viewports without stripping motion:
```javascript
ScrollTrigger.matchMedia({
  // Desktop: complex pinning and scrubbing
  '(min-width: 1024px)': function() {
    gsap.timeline({ scrollTrigger: { trigger: '.hero-pin', pin: true, scrub: 1 } })
        .to('.hero-panel', { xPercent: -100 });
  },
  // Mobile: streamlined transform without sticky pinning
  '(max-width: 1023px)': function() {
    gsap.to('.hero-panel', { opacity: 1, y: 0, scrollTrigger: { trigger: '.hero-pin', start: 'top 85%' } });
  }
});
```
