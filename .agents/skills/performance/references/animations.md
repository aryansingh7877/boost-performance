# Evidence-Based Animation Performance Reference Manual

## 1. Prime Rule: Never Blame Animations Without Profiler Evidence

Choreography and motion are fundamental to modern creative web products.
- **Rule**: **Do NOT automatically blame animations, CSS transitions, Framer Motion, or GSAP for poor performance.**
- Attribute a performance issue to an animation only when direct profiler evidence proves it:
  - DevTools Performance panel shows frame drops (<60 FPS) precisely aligned with animation ticks.
  - "Forced Synchronous Layout" (Layout Thrashing) warnings appear during animation frames.
  - Call-tree traces identify animation callbacks as the source of Long Tasks (>50ms).
- **Preserve Visual Motion**: Never propose disabling, removing, or shortening animations as a performance fix. Always optimize the underlying execution mechanics to run at display refresh rates.

---

## 2. The Browser Rendering Pipeline

Understanding browser render phases determines whether an animation runs on the GPU compositor at refresh rate or stutters on the main thread:

1. **JavaScript**: Style calculation and property tween calculations.
2. **Style Calculation**: Recalculating CSS rules matching DOM elements.
3. **Layout (Reflow)**: Computing element spatial geometry (`width`, `height`, `top`, `left`). *Expensive: triggers geometric recalculations across parent and sibling nodes.*
4. **Paint**: Rasterizing vector shapes, typography, and images into bitmap layers. *Expensive: re-paints affected surface areas.*
5. **Composite**: Combining pre-rendered layers on the GPU. *Extremely cheap: hardware-accelerated, runs on a separate compositor thread.*

---

## 3. The Golden Rule of Web Motion: Composited Properties Only

**Only animate properties handled exclusively at the GPU Composite stage:**
- `transform`: `translate3d(x, y, z)`, `translateX()`, `translateY()`, `scale()`, `rotate()`
- `opacity`
- `filter`: Hardware-accelerated, but heavier than transform/opacity; use judiciously.

**Strict Anti-pattern: Never animate layout-triggering properties:**
- `top`, `left`, `right`, `bottom`
- `width`, `height`
- `margin`, `padding`
- `border-width`

### Migration Examples (Zero Visual Change):
```css
/* ❌ Jank-inducing layout reflow on every frame */
.card {
  top: 50px;
  width: 300px;
}

/* ✅ 60fps GPU-composited transformation */
.card {
  transform: translateY(50px) scaleX(1.5);
  transform-origin: left center;
}
```

---

## 4. Animation Loops & Timer Synchronization

- **`requestAnimationFrame`**: Always synchronize manual JavaScript animation loops with `window.requestAnimationFrame`. It synchronizes with display v-sync refresh and pauses when the tab is backgrounded.
- **Never use `setInterval` or `setTimeout` for visual motion.** Fixed timers drift out of sync with hardware refresh cycles, cause visual stutter, and continue burning CPU cycles in background tabs.

---

## 5. Eliminating Layout Thrashing in Hot Callbacks

Never interleave DOM geometry reads (`getBoundingClientRect()`, `offsetHeight`, `scrollTop`) with DOM style writes (`element.style.transform = ...`) within the same loop. Interleaving reads and writes forces the browser into repeated, synchronous layout reflows ("layout thrashing").
- **Solution**: Batch all geometry reads first, then execute all style writes together on the next animation frame.

---

## 6. Targeted `will-change` Discipline

- Apply `will-change: transform` or `will-change: opacity` only to elements currently animating or immediately about to animate.
- **Strict Anti-pattern**: Never set `will-change: all` or apply `will-change` permanently across large DOM trees. Excessive compositor layer creation exhausts GPU VRAM, particularly on mobile devices. Remove the property once the animation sequence completes.

---

## 7. Reduced Motion Support

Honor `prefers-reduced-motion: reduce`:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```
This respects user accessibility requirements while conserving CPU/battery on constrained hardware.
