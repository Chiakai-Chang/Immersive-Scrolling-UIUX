# Immersive Motion Modules (M01–M10)

Production-ready patterns for the 10 core modules. Always wrap ScrollTrigger setups inside `gsap.matchMedia()` and honor `prefers-reduced-motion: reduce`.

---

## M01: Cinematic Hero

Combines an immediate entrance timeline with a scrubbed background scale/parallax on scroll.

```js
// Entrance
gsap.from('.hero-line', {
  yPercent: 110,
  opacity: 0,
  duration: 0.9,
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

## Web Motion Playground Pattern

When building a reusable playground for M01–M10:
1. Store each module as a self-contained descriptor: `{ id, title, category, paramsSchema, setup(container, params), codeSnippet }`.
2. Render a live preview stage + parameter sliders (`scrub`, `duration`, `ease`, `intensity`) that re-run `ctx.revert()` (`gsap.context()`) and re-initialize `setup()` cleanly on change.
3. Include a **Reduced Motion Toggle** and **ScrollTrigger Markers Toggle** directly in the playground toolbar for instant verification.
