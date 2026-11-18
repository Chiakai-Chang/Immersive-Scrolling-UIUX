# Immersive Motion Modules (M01–M10)

Production-ready patterns for the 10 core modules. Always wrap ScrollTrigger setups inside `gsap.matchMedia()` and honor `prefers-reduced-motion: reduce`.

---

## M01: Cinematic Hero

Combines an immediate, **LCP-safe** entrance (animate `yPercent` inside `overflow: hidden` without setting initial `opacity: 0` on the LCP `<h1>`) with a scrubbed background scale/parallax on scroll.

```js
// LCP-safe Entrance (parent .line-wrap has overflow: hidden)
gsap.from('.hero-line', {
  yPercent: 105,
  duration: 0.85,
  stagger: 0.08,
  ease: 'power3.out',
});

// Scroll exit parallax
gsap.to('.hero-bg', {
  scale: 1.15,
  yPercent: 18,
  ease: 'none',
  scrollTrigger: {
    trigger: '.hero',
    start: 'top top',
    end: 'bottom top',
    scrub: true,
  },
});
```

---

## M02: Parallax Image (Dual Option)

**Option A — Pure CSS Scroll-Driven (Preferred when no pinning is needed):**

```css
@supports (animation-timeline: view()) {
  @media (prefers-reduced-motion: no-preference) {
    .parallax-img {
      animation: parallax-shift linear both;
      animation-timeline: view();
      animation-range: entry 0% exit 100%;
    }
    @keyframes parallax-shift {
      from { transform: translateY(-8%) scale(1.12); }
      to   { transform: translateY(8%)  scale(1.12); }
    }
  }
}
```

**Option B — GSAP `data-speed` layers:**

```js
gsap.utils.toArray('[data-speed]').forEach((el) => {
  const speed = parseFloat(el.dataset.speed) || 0.2;
  gsap.to(el, {
    yPercent: speed * 100,
    ease: 'none',
    scrollTrigger: {
      trigger: el.closest('section') || el,
      start: 'top bottom',
      end: 'bottom top',
      scrub: true,
    },
  });
});
```

---

## M03: Accessible Text Reveal

Keep screen readers intact by placing the full string in `aria-label` and hiding split spans.

```html
<h2 class="reveal-heading" aria-label="Crafted for cinematic motion">
  <span class="line-wrap"><span class="line" aria-hidden="true">Crafted for</span></span>
  <span class="line-wrap"><span class="line" aria-hidden="true">cinematic motion</span></span>
</h2>
```

```css
.line-wrap { display: block; overflow: hidden; }
.line { display: inline-block; will-change: transform; }
```

```js
gsap.from('.reveal-heading .line', {
  yPercent: 105,
  duration: 0.85,
  stagger: 0.1,
  ease: 'power4.out',
  scrollTrigger: {
    trigger: '.reveal-heading',
    start: 'top 80%',
    once: true,
  },
});
```

---

## M04: Image Mask Transition (`clip-path`)

GPU-friendly `inset()` reveal scrubbed or triggered on scroll:

```js
gsap.fromTo('.mask-media',
  { clipPath: 'inset(18% 12% 18% 12% round 16px)', scale: 1.12 },
  {
    clipPath: 'inset(0% 0% 0% 0% round 0px)',
    scale: 1,
    ease: 'none',
    scrollTrigger: {
      trigger: '.mask-section',
      start: 'top 75%',
      end: 'top 20%',
      scrub: 0.6,
    },
  }
);
```

---

## M05: Sticky / Pinned Storytelling

Keep the stage pinned while chapters crossfade sequentially on a single scrubbed timeline.

```js
const chapters = gsap.utils.toArray('.story-chapter');
const tl = gsap.timeline({
  scrollTrigger: {
    trigger: '.story-stage',
    start: 'top top',
    end: () => `+=${chapters.length * window.innerHeight}`,
    pin: true,
    scrub: 0.8,
    invalidateOnRefresh: true,
  },
});

chapters.forEach((ch, i) => {
  if (i > 0) {
    tl.to(chapters[i - 1], { opacity: 0, y: -24, duration: 0.4 })
      .fromTo(ch, { opacity: 0, y: 24 }, { opacity: 1, y: 0, duration: 0.4 }, '<0.1');
  }
});
```

---

## M06: Horizontal Scroll + `containerAnimation`

Use `overflow-x: clip` on the outer page wrapper so `pin: true` works cleanly.

```js
const track = document.querySelector('.h-track');

const horizontalTween = gsap.to(track, {
  x: () => -(track.scrollWidth - window.innerWidth),
  ease: 'none',
  scrollTrigger: {
    trigger: '.h-section',
    pin: true,
    scrub: 1,
    end: () => `+=${track.scrollWidth - window.innerWidth}`,
    invalidateOnRefresh: true,
  },
});

// Nested trigger inside the horizontal track:
gsap.utils.toArray('.h-card').forEach((card) => {
  gsap.from(card, {
    opacity: 0.3,
    scale: 0.92,
    scrollTrigger: {
      trigger: card,
      containerAnimation: horizontalTween,
      start: 'left 85%',
      end: 'left 45%',
      scrub: true,
    },
  });
});

// WCAG 2.2 Keyboard Accessibility: scroll focused card into view on Tab
track.addEventListener('focusin', (e) => {
  const card = e.target.closest('.h-card');
  const st = horizontalTween.scrollTrigger;
  if (!card || !st) return;
  const progress = card.offsetLeft / Math.max(1, track.scrollWidth - window.innerWidth);
  window.scrollTo({ top: st.start + progress * (st.end - st.start), behavior: 'auto' });
});
```

---

## M07: Three.js 3D Camera & Object Scrub

Keep `<canvas>` fixed behind DOM sections; DOM sections control scroll height, and `ScrollTrigger` scrubs camera/mesh waypoints.

```js
import * as THREE from 'three';

let isCanvasInView = true;
const observer = new IntersectionObserver(([entry]) => {
  isCanvasInView = entry.isIntersecting;
});
observer.observe(canvas);

renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

gsap.ticker.add(() => {
  if (isCanvasInView) renderer.render(scene, camera);
});

const camTl = gsap.timeline({
  scrollTrigger: {
    trigger: '.experience-wrapper',
    start: 'top top',
    end: 'bottom bottom',
    scrub: 1,
  },
});

camTl
  .to(camera.position, { x: 2, y: 0.5, z: 4, ease: 'none' })
  .to(heroMesh.rotation, { y: Math.PI * 1.5, ease: 'none' }, 0);
```

Always provide a `disposeScene()` helper calling `geometry.dispose()`, `material.dispose()`, and `renderer.dispose()`.

---

## M08: Magnetic Cursor (`gsap.quickTo`)

Activate only on `@media (pointer: fine)` and `prefers-reduced-motion: no-preference`.

```js
if (window.matchMedia('(pointer: fine) and (prefers-reduced-motion: no-preference)').matches) {
  document.querySelectorAll('[data-magnetic]').forEach((btn) => {
    const xTo = gsap.quickTo(btn, 'x', { duration: 0.35, ease: 'power3.out' });
    const yTo = gsap.quickTo(btn, 'y', { duration: 0.35, ease: 'power3.out' });

    btn.addEventListener('mousemove', (e) => {
      const rect = btn.getBoundingClientRect();
      const dx = (e.clientX - rect.left - rect.width / 2) * 0.35;
      const dy = (e.clientY - rect.top - rect.height / 2) * 0.35;
      xTo(dx);
      yTo(dy);
    });

    btn.addEventListener('mouseleave', () => {
      xTo(0);
      yTo(0);
    });
  });
}
```

---

## M09: Particle / GLSL Shader Background

- Keep particle count reasonable (`1500–4000` points with `BufferGeometry` + `PointsMaterial` or custom `ShaderMaterial`).
- Pass `uScroll` (0..1 from ScrollTrigger `onUpdate: (self) => uniforms.uScroll.value = self.progress`) and smoothed `uMouse` (`gsap.quickTo` on a uniform vector).
- Pause `uTime` updates and `renderer.render()` when off-screen or when `prefers-reduced-motion: reduce`.

---

## M10: Morphing / Scene Bridge Transitions

- **CSS/SVG:** Animate `clip-path: circle(0% at 50% 50%)` -> `circle(150% at 50% 50%)` or SVG `<path>` `d` attribute via GSAP `scrub` between contrasting light/dark sections.
- **Background Theme Crossfade:** Use `ScrollTrigger` `onEnter` / `onLeaveBack` to tween CSS custom properties (`--bg`, `--fg`) on `document.documentElement` over `0.5s` for editorial color shifts between chapters.

---

## M11: Pseudo-3D Sprite Stacking, Multi-Camera Rig & Diegetic HUD

Inspired by ultra-lightweight (<32KB) 2D Canvas 3D experiences (e.g., *Fly*): render volumetric 3D objects, terrain depth, and seamless camera transitions (`Overview` ↔ `Chase` ↔ `Cockpit HUD`) without loading a 450KB+ WebGL engine.

### 1. Sprite Stacking Core (Volumetric Slices on 2D Canvas)

Pre-render or define horizontal slices bottom-to-top and draw them with a vertical offset proportional to camera pitch:

```js
function drawSpriteStack(ctx, slices, x, y, angle, pitchTilt = 1, sliceGap = 2.5, scale = 1) {
  slices.forEach((sliceCanvas, i) => {
    ctx.save();
    // Offset upward along screen Y to create 3D extrusion
    ctx.translate(x, y - i * sliceGap * pitchTilt * scale);
    ctx.rotate(angle);
    ctx.scale(scale, scale * pitchTilt);
    ctx.drawImage(sliceCanvas, -sliceCanvas.width / 2, -sliceCanvas.height / 2);
    ctx.restore();
  });
}
```

- **Depth & Shading:** Slightly darken lower slices (`i = 0..n/2`) during pre-bake for instant ambient occlusion.
- **Painter's Sort:** Sort world objects by `cameraRelativeY` before drawing so distant islands/clouds render behind foreground objects.

### 2. Multi-Camera Interpolation (Frame-Rate Independent Lerp)

Switch smoothly between `topDown`, `chase`, and `hud` camera presets using exponential damping (immune to 60Hz vs 120Hz differences):

```js
const CAM_PRESETS = {
  topDown: { pitchTilt: 0.95, sliceGap: 1.8, zoom: 0.85, hudOpacity: 0 },
  chase:   { pitchTilt: 0.52, sliceGap: 3.2, zoom: 1.25, hudOpacity: 0.35 },
  hud:     { pitchTilt: 0.28, sliceGap: 4.0, zoom: 1.75, hudOpacity: 1 },
};

const damp = (curr, target, lambda, dt) =>
  curr + (target - curr) * (1 - Math.exp(-lambda * dt));

// Inside gsap.ticker.add((time, deltaTime) => ...)
const dt = Math.min(deltaTime / 1000, 0.05);
cam.pitchTilt = damp(cam.pitchTilt, preset.pitchTilt, 6, dt);
cam.zoom      = damp(cam.zoom,      preset.zoom,      6, dt);
```

### 3. Diegetic HUD & Performance Guardrails

- **HiDPI Crispness:** Size the canvas backing store by `dpr = Math.min(window.devicePixelRatio, 2)` and call `ctx.setTransform(dpr, 0, 0, dpr, 0, 0)` on resize so pixel/vector HUD lines stay razor-sharp on Retina screens.
- **Diegetic Telemetry:** Map scroll progress or pointer velocity to crisp vector HUD elements (compass heading, depth/altitude scale, lock-on reticle) instead of generic floating cards.
- **Offscreen Pre-baking:** Draw static pixel/vector slices onto `OffscreenCanvas` (or hidden `<canvas>`) once at startup; never allocate gradients or paths inside the per-frame loop.
- **Reduced Motion:** Snap immediately to the target camera preset (`lambda = Infinity`) and render a clean static frame when `prefers-reduced-motion: reduce` is active.

---

## M12: Image-Sequence / Frame Scrub (`<canvas>`)

Use a `<canvas>` image sequence (rather than scrubbing `<video>.currentTime`, which stutters on reverse scroll due to keyframe decoding) for Apple-style product spins or teardowns:

```js
const frameCount = 60;
const images = Array.from({ length: frameCount }, (_, i) => {
  const img = new Image();
  img.src = `/frames/frame-${String(i + 1).padStart(3, '0')}.webp`;
  return img;
});

const state = { frame: 0 };
const render = () => {
  const img = images[Math.round(state.frame)];
  if (img?.complete && img.naturalWidth > 0) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
  }
};

images[0].onload = render; // Paint poster frame immediately

gsap.to(state, {
  frame: frameCount - 1,
  snap: 'frame',
  ease: 'none',
  scrollTrigger: { trigger: '.sequence-section', start: 'top top', end: '+=2000', pin: true, scrub: 0.5 },
  onUpdate: render,
});
```

- **Reduced Motion / Slow Network:** Render `images[0]` (poster frame) statically and skip `ScrollTrigger` pinning.

---

## Web Motion Playground Pattern

When building a reusable playground for M01–M12:
1. Store each module as a self-contained descriptor: `{ id, title, category, paramsSchema, setup(container, params), codeSnippet }`.
2. Render a live preview stage + parameter sliders (`scrub`, `duration`, `ease`, `intensity`) that re-run `ctx.revert()` (`gsap.context()`) and re-initialize `setup()` cleanly on change.
3. Include a **Reduced Motion Toggle** and **ScrollTrigger Markers Toggle** directly in the playground toolbar for instant verification.

---

## React / Next.js (`useGSAP`) Lifecycle Pattern

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

### React Three Fiber (`@react-three/fiber`) + GSAP Bridge

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

- **Asset & Dev Hygiene:** Convert M12 frame sequences and textures to **WebP/AVIF** (e.g., via `sharp`) to protect LCP; strip debug tools (`leva`, `r3f-perf`) in production builds.

---

## Glass Overlay, View Transitions & State Completeness

### 1. Frosted / Liquid Glass over Dynamic Backdrops

```css
.glass-panel {
  background: rgba(18, 22, 30, 0.62);
  backdrop-filter: blur(18px) saturate(160%);
  -webkit-backdrop-filter: blur(18px) saturate(160%);
  border: 1px solid rgba(255, 255, 255, 0.16);
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.18);
}

@media (prefers-reduced-transparency: reduce) {
  .glass-panel {
    background: #12161e;
    backdrop-filter: none;
    -webkit-backdrop-filter: none;
  }
}
```

### 2. View Transitions API (Seamless State / Filter Morph)

```js
function transitionState(updateDom) {
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (!document.startViewTransition || reduced) {
    updateDom();
    return;
  }
  document.startViewTransition(updateDom);
}
```

### 3. Loading / Error Fallback for 3D & Frame Sequences

Always render a visible `[data-stage-status="loading"]` poster/skeleton until assets decode, and switch to `[data-stage-status="fallback"]` (static hero image) on WebGL context loss or asset load error.
