# Architecture, Frameworks & WebGL Optimization

Load this reference when integrating scroll choreography into **React / Next.js / React Three Fiber**, optimizing **Three.js draw calls & shader compilation**, or implementing **adaptive hardware tiering**.

---

## 1. React / Next.js (`useGSAP`) Lifecycle Pattern

In React 18/19 StrictMode or SPA route transitions, always use `@gsap/react`'s `useGSAP()` (or `gsap.context()`) so ScrollTriggers revert cleanly on unmount:

```tsx
import { useRef } from 'react';
import gsap from 'gsap';
import { useGSAP } from '@gsap/react';
import { ScrollTrigger } from 'gsap/ScrollTrigger';

gsap.registerPlugin(ScrollTrigger);

export function StorySection() {
  const scopeRef = useRef(null);

  useGSAP(() => {
    const mm = gsap.matchMedia();
    mm.add('(min-width: 1024px) and (prefers-reduced-motion: no-preference)', () => {
      gsap.to('.panel', {
        yPercent: -15,
        scrollTrigger: { trigger: '.panel', scrub: true },
      });
    });
  }, { scope: scopeRef });

  return <section ref={scopeRef}>{/* ... */}</section>;
}
```

---

## 2. React Three Fiber (`@react-three/fiber`) + GSAP Bridge

In R3F, `<Canvas>` owns the WebGL render loop—never call `renderer.render()` inside `gsap.ticker`. Instead, mutate a shared ref/store from `ScrollTrigger` and read it in `useFrame()`:

```tsx
const scrollProgress = useRef(0);

useGSAP(() => {
  ScrollTrigger.create({
    trigger: '.r3f-story',
    start: 'top top',
    end: 'bottom bottom',
    scrub: true,
    onUpdate: (self) => { scrollProgress.current = self.progress; },
  });
});

// Inside <Canvas> child component:
useFrame(({ camera }) => {
  camera.position.z = THREE.MathUtils.lerp(6, 2.5, scrollProgress.current);
});
```

---

## 3. Zero-Stutter Shader Warmup & Texture Atlas

- **Shader Warmup (`gl.compileAsync`):** Pre-compile shaders during preloader or idle time (`await renderer.compileAsync?.(scene, camera)`) so scrolling into a new 3D section has zero first-frame hitch. Skip on `LOW` tier devices to prevent `webglcontextlost`.
- **Texture Atlas (N Images -> 1 Draw Call):** Draw gallery images onto a single `OffscreenCanvas` grid (`cols × rows`, `2px` padding) at startup and upload one `new THREE.CanvasTexture(canvas)` instead of binding dozens of separate textures.
- **Asset & Dev Hygiene:** Convert frame sequences and textures to **WebP/AVIF** (e.g., via `sharp`) to protect LCP; strip debug tools (`leva`, `r3f-perf`) in production builds.

---

## 4. Adaptive Hardware Performance Tiering (`HIGH / MEDIUM / LOW`)

Gate DPR, shadows, antialiasing, and particle density by hardware capability—not viewport width alone:

```js
export function detectPerformanceTier() {
  const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
  const cores = navigator.hardwareConcurrency || 8;
  const ram = navigator.deviceMemory || 8;

  if (ram <= 4 || (isMobile && cores <= 4)) {
    return { tier: 'LOW', dpr: 1, shadows: false, antialias: false, particleScale: 0.3 };
  }
  if (isMobile || cores <= 4) {
    return { tier: 'MEDIUM', dpr: 1.5, shadows: false, antialias: true, particleScale: 0.6 };
  }
  return { tier: 'HIGH', dpr: 2, shadows: true, antialias: true, particleScale: 1.0 };
}
```

Expose a runtime `downgradeTier()` callback if frame times spike during heavy WebGL scenes.

---

## 5. Velocity-Driven Infinite Scroll (for Looping 3D Galleries)

When building an infinite 3D gallery without a document scrollbar, accumulate input impulses with frame-rate-independent exponential friction (`Math.pow(0.94, dtFactor)`) and expose `velocity` for subtle skew/tilt reactivity:

```js
let offset = 0, velocity = 0, pending = 0;
window.addEventListener('wheel', (e) => { pending += e.deltaY * 0.01; }, { passive: true });

// Inside gsap.ticker.add((_, deltaTime) => ...):
const dtFactor = Math.min((deltaTime || 16) / 16.67, 2);
velocity += pending;
pending = 0;
velocity *= Math.pow(0.94, dtFactor);
if (Math.abs(velocity) < 0.0001) velocity = 0;
offset += velocity;
```
