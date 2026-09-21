# Mobile Web Performance Engineering Reference Manual

## 1. Mobile-First Performance Baseline

Mobile devices are defined by three physical constraints: constrained CPU/GPU thermal ceilings, limited memory (VRAM/RAM), and variable network conditions. Techniques that perform adequately on desktop hardware (uncapped `devicePixelRatio`, multi-megabyte hero videos, dense unbatched scroll triggers) degrade into immediate frame drops, thermal throttling, and battery drain on mobile devices.

---

## 2. Thermal Throttling & GPU Power Ceilings

Under sustained high CPU or GPU workloads, mobile operating systems dynamically reduce processor clock frequencies to manage internal thermal rise:
- **The Thermal Trap**: A 3D canvas or complex GSAP animation may maintain 60 FPS for the first 20 seconds, but drop to 25 FPS as the mobile SOC throttles clock speeds.
- **Remediation**:
  - Clamp device pixel ratio (`dpr={[1, 1.5]}` or `dpr={[1, 2]}`) to prevent shading millions of redundant pixels (see `threejs-webgl.md`).
  - Pause offscreen WebGL rendering and video decoding via `IntersectionObserver` (see `video.md`).
  - Restrict heavy `backdrop-filter: blur()` effects on mobile viewports.

---

## 3. Responsive Media Payloads

- **Adaptive Resolution**: Never serve a 2560px desktop image to a 390px mobile viewport. Use responsive `srcset` and accurate `sizes` to ensure mobile devices download resolution-matched assets (see `images.md`).
- **Video Strategy**: On mobile cellular networks, multi-megabyte hero videos can stall bandwidth. Ensure `preload="metadata"` is set, provide an optimized WebM stream, and ensure `muted playsinline` attributes are present to prevent disruptive fullscreen native player takeover.

---

## 4. Touch & Scroll Responsiveness

- **Passive Event Listeners**: Always pass `{ passive: true }` to `touchstart`, `touchmove`, and `wheel` listeners that do not invoke `event.preventDefault()`. This allows the browser compositor to initiate scrolling immediately without blocking on JavaScript execution.
- **Tap Delay Elimination**: Ensure the viewport meta tag includes `width=device-width, initial-scale=1` to disable the legacy 300ms double-tap zoom delay.

---

## 5. Network Latency & Mobile Profiles

- Mobile networks exhibit higher latency (RTT) and jitter than wired connections. A resource discovery waterfall that takes 100ms on desktop broadband can take 800ms+ on a mobile 4G connection.
- **Testing Standard**: When live testing tools support emulation profiles, always prefer testing under a **Mobile / Throttled 4G** profile rather than unthrottled desktop.
