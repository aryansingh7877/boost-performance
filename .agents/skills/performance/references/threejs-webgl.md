# Evidence-Based Three.js & WebGL Performance Reference Manual

## 1. Prime Rule: Never Remove 3D Scenes Without Empirical Proof

Three.js and WebGL scenes deliver immersive visual experiences that define modern creative websites.
- **Rule**: **Do NOT automatically remove, disable, or simplify 3D scenes.**
- Only attribute a performance bottleneck to Three.js/R3F when evidence directly supports it:
  - GPU profiling shows frame drops (<60 FPS) specifically during WebGL draw calls or shader execution.
  - Device thermal throttling causes frame rates to collapse after sustained interaction.
  - The canvas renders continuously in the background when completely out of the viewport.
  - VRAM consumption climbs monotonically due to missing resource disposal.
- **Differentiate CPU vs. GPU Bottlenecks**: Do not optimize JavaScript CPU code when the real bottleneck is GPU fill-rate, and do not lower texture resolutions when the bottleneck is main-thread asset parsing.

---

## 2. Isolating the Three.js Bottleneck Layer

| Bottleneck Layer | Symptoms & Evidence | Targeted Remediation |
| :--- | :--- | :--- |
| **Network & Asset Loading** | Late 3D scene appearance; large model downloads block LCP. | Compress models with Draco or Meshopt; size textures to power-of-two; lazy-load non-hero canvases. |
| **JS Initialization** | Long Tasks (>50ms) during Three.js scene setup or shader compilation. | Pre-compile materials with `renderer.compile()`; initialize heavy scenes off the critical hydration path. |
| **Continuous Rendering** | High CPU/battery drain while page is idle or canvas is offscreen. | Implement `frameloop="demand"` or pause the RAF loop via `IntersectionObserver` when offscreen. |
| **GPU Fill-Rate / Shading** | Frame drops on high-density Retina/mobile screens. | Clamp device pixel ratio (`dpr={[1, 2]}`); reduce full-screen post-processing passes. |
| **Draw Call Overhead** | High CPU overhead in render loop; thousands of individual meshes. | Consolidate meshes with `THREE.InstancedMesh`; merge static geometries with `BufferGeometryUtils`. |
| **GPU Memory (VRAM) Leaks** | Tab crashes or progressive degradation over navigation sessions. | Explicitly invoke `.dispose()` on geometries, materials, textures, and render targets. |

---

## 3. Device Pixel Ratio (DPR) Clamping (#1 Mobile & Retina Win)

High-density mobile and Retina displays frequently report `devicePixelRatio` values of 3.0 or 4.0. Rendering a full-viewport WebGL canvas at 3x DPR requires shading **9 times** the pixel count of a standard 1x display, placing massive strain on GPUs for minimal perceptible difference:

### React Three Fiber (R3F):
```tsx
// Clamp DPR strictly between 1 and 2
<Canvas dpr={[1, 2]} camera={{ position: [0, 0, 5], fov: 45 }}>
  {/* scene content */}
</Canvas>
```
*Note: An R3F `<Canvas>` without an explicit `dpr` prop defaults to raw `window.devicePixelRatio`. Always supply `dpr={[1, 2]}`.*

### Vanilla Three.js:
```javascript
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
```

---

## 4. WebGL Resource Disposal (VRAM Leak Prevention)

Geometries, materials, textures, and render targets live in GPU VRAM and are **never** garbage-collected by the JavaScript engine automatically:

```javascript
useEffect(() => {
  const geometry = new THREE.BoxGeometry();
  const texture = new THREE.TextureLoader().load('/textures/surface.webp');
  const material = new THREE.MeshStandardMaterial({ map: texture });
  const mesh = new THREE.Mesh(geometry, material);

  scene.add(mesh);

  return () => {
    scene.remove(mesh);
    geometry.dispose();
    material.dispose();
    texture.dispose();
  };
}, []);
```
In R3F, unmounted meshes inside standard component trees are disposed automatically by R3F's reconciler, but manually instantiated textures, geometries, or custom render targets must be explicitly disposed.

---

## 5. Render Loop Control & Viewport Visibility

A WebGL canvas continuously rendering 60 times a second while scrolled far out of view wastes battery, heats devices, and steals main-thread capacity:

### On-Demand Rendering in R3F:
```tsx
// Only render when state, camera, or props change
<Canvas frameloop="demand">
  {/* scene */}
</Canvas>
```

### Intersection-Based Pausing:
Pause rendering when the canvas is not in the active viewport:
```javascript
const canvasObserver = new IntersectionObserver(([entry]) => {
  if (entry.isIntersecting) {
    renderer.setAnimationLoop(animate);
  } else {
    renderer.setAnimationLoop(null);
  }
}, { threshold: 0.05 });

canvasObserver.observe(renderer.domElement);
```

---

## 6. Draw Call Reduction & Instancing

- **`InstancedMesh`**: When rendering multiple copies of identical geometry (particles, foliage, repeating UI meshes), use `THREE.InstancedMesh` (or `@react-three/drei` `<Instances>`). This collapses hundreds or thousands of individual draw calls into a single GPU call.
- **Geometry Merging**: For non-animated static meshes sharing the same material, merge geometries into a single buffer using `BufferGeometryUtils.mergeGeometries`.

---

## 7. Lighting & Shadow Map Discipline

- **Shadow Maps**: Real-time shadows require an additional render pass per shadow-casting light. Restrict shadow casting to 1 primary directional light. Clamp shadow map resolutions to 1024px or 2048px maximum (`light.shadow.mapSize.set(1024, 1024)`).
- **Light Culling**: Avoid multiple dynamic point lights. For static environments, bake lighting into textures.
- **Material Efficiency**: Use `MeshBasicMaterial` or `MeshLambertMaterial` for distant or secondary meshes where physically-based PBR specular reflections are imperceptible.
