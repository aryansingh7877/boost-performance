# Conditional Video Performance Reference Manual

## 1. Prime Rule: Never Delete Video to Manipulate Scores

Video is a primary visual storytelling, branding, and product presentation medium.
- **Never delete, disable, or replace a video with a static placeholder merely to boost a Lighthouse score.**
- An optimization that improves metrics by breaking autoplay, removing looping, or degrading visual quality is a **regression**.
- Always optimize video delivery, stream negotiation, and runtime execution while strictly preserving playback behavior.

---

## 2. Conditional Video Decision Engine

Do **not** apply identical blanket rules to every video. Evaluate each video element across these technical dimensions:

| Question | Technical Implication | Recommended Engineering Strategy |
| :--- | :--- | :--- |
| **Is it Above the Fold?** | Competes with initial paint & LCP resources. | Supply an optimized poster frame (`poster="..."`), set `preload="metadata"`, and encode at a balanced bitrate. |
| **Does it contribute to LCP?** | Video poster or first video frame is the LCP candidate. | Prioritize the poster image; ensure poster dimensions match container aspect ratio. |
| **Is it Decorative / Background?** | No user controls; loops continuously without audio. | Add `autoplay muted playsinline loop`. Target 1.5–2.5 Mbps bitrate. Pause offscreen via `IntersectionObserver`. |
| **Is it Interactive / Content?** | User initiates playback (e.g. tutorial, product demo). | Set `preload="none"`. Load stream only on user interaction or hover intent. |
| **Is it Below the Fold?** | Hidden during initial navigation. | Defer stream loading using `IntersectionObserver`. Set `preload="none"`. |
| **Is Full Stream Downloaded?** | Browser buffers entire multi-megabyte file upfront. | Enforce `preload="metadata"` to download only duration/dimensions headers. |

---

## 3. Above-the-Fold (Hero Video) Strategy

For critical above-the-fold videos:
- **Fast-Rendering Poster**: Provide a modern AVIF or WebP poster image. The browser renders the poster immediately, avoiding blank/black boxes and eliminating layout shifts (CLS).
- **Metadata Preloading**: Use `preload="metadata"`. This allows the browser to parse video headers and allocate the correct canvas dimensions without buffering the entire media stream upfront.
- **Dual Format Sources**: Offer WebM (VP9/AV1) ahead of MP4 (H.264). WebM delivers 30–50% bandwidth savings for looping video backgrounds:
  ```html
  <video
    autoplay
    muted
    playsinline
    loop
    preload="metadata"
    poster="/media/hero-poster.webp"
    class="hero-video"
  >
    <source src="/media/hero.webm" type="video/webm">
    <source src="/media/hero.mp4" type="video/mp4">
  </video>
  ```

---

## 4. Below-the-Fold & Offscreen Optimization

A video that decodes and plays continuously while scrolled far out of view wastes significant CPU/GPU resources and battery:

### Intersection-Based Playback Pausing:
Pause decoding when the video leaves the active viewport, and resume when it re-enters:
```javascript
const videoObserver = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    const video = entry.target;
    if (entry.isIntersecting) {
      video.play().catch(() => {/* handle browser autoplay policies */});
    } else {
      video.pause();
    }
  });
}, { threshold: 0.1 });

document.querySelectorAll('video[autoplay]').forEach((video) => {
  videoObserver.observe(video);
});
```

### Below-the-Fold Deferred Loading:
For non-hero videos located significantly down the page, defer setting `src` until the element approaches within 300px of the viewport.

---

## 5. Mobile Autoplay Compliance

Mobile operating systems (iOS Safari, Android Chrome) enforce strict power and data restrictions:
- Mobile browsers will **refuse to autoplay** unless **both** `muted` and `playsinline` attributes are present.
- Missing `playsinline` on iOS causes the video to launch in a disruptive fullscreen system player.
- Never remove `muted` or `playsinline` when refactoring video elements.

---

## 6. Anti-Patterns to Strictly Avoid

- **Never blindly replace video with static imagery.**
- **Never lazy-load an above-the-fold hero video's container** behind client JavaScript if it forms the main visual presentation.
- **Never preload the full stream (`preload="auto"`)** on multiple video elements simultaneously, as this starves critical JavaScript and CSS network requests.
