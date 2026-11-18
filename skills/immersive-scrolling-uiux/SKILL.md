---
name: immersive-scrolling-uiux
description: Use when building or upgrading scroll-driven websites, scrollytelling landing pages, parallax or pinned sections, horizontal scroll galleries, Three.js or WebGL scroll scenes, or interactive motion playgrounds.
---

# Immersive Scrolling UI/UX

## Overview

Cinematic web experiences are not isolated animations—they are **choreographed timelines** where background scale, foreground parallax, mask reveals, typography, and 3D camera motion share a unified rhythm.

**Core rule:** Define a Motion Specification before writing code, unify all render loops on a single ticker, and verify each section in a real browser before building the next.

## When to Use

- Building scrollytelling landing pages, product launches, agency/studio portfolios, or editorial narratives.
- Combining **GSAP ScrollTrigger**, **Lenis**, **Three.js / WebGL**, or **CSS Scroll-driven Animations** (`animation-timeline`).
- Creating a **Web Motion Playground** or reusable motion module library (M01–M10).

**When NOT to use:** Standard SaaS dashboards, data tables, or CRUD forms where scroll-jacking or pinned sections hurt task efficiency. Pair with `frontend-design` or `design-taste-frontend` for static visual direction, and `motion-choreography` for standard UI state transitions.

## Technology Decision Matrix

Choose the lightest tool that satisfies the choreography:

| Effect Requirement | Use | Why |
|---|---|---|
| Simple progress bar, card fade/scale on scroll | **CSS `animation-timeline: scroll()` / `view()`** | Zero JS, compositor-thread smooth, Baseline 2026 |
| Pinned sections, scrubbed timelines, horizontal scroll | **GSAP + ScrollTrigger** | Deterministic pin-spacers, `scrub`, `containerAnimation` |
| Smooth wheel inertia | **Lenis** (synced to `gsap.ticker`) | Decouples wheel smoothing without breaking `ScrollTrigger` |
| Lightweight 3D objects, multi-camera views, HUDs (<32KB) | **2D Canvas Pseudo-3D (Sprite Stacking)** | Zero Three.js bundle cost; instant FCP, volumetric look |
| Full PBR lighting, 3D `.glb` models, GLSL shaders | **Three.js** (driven by ScrollTrigger) | Full WebGL control; keep DOM as the scroll height driver |
| Pointer-following / magnetic elements | **`gsap.quickTo()`** | Reuses a single tween; never spawn new tweens per `mousemove` |

## Mandatory Workflow

Do **not** generate an entire multi-section site in one shot. Follow this 5-step loop:

1. **Design Brief:** Lock theme, emotional tone, typography, palette, and 1–2 memorable visual moments.
2. **Motion Specification:** For each section, write a compact spec (see template below) BEFORE coding.
3. **Section-by-Section Implementation:** Build HTML + CSS layout first, then wire motion for **one section at a time**.
4. **Browser Validation:** Inspect `ScrollTrigger` markers, 60fps compositor performance, mobile touch behavior, and `prefers-reduced-motion` after each section.
5. **Visual Refinement:** Tune easing, scrub lag, spacing, and stagger overlap.

### Motion Specification Template

Write this per section before implementing:

```yaml
section: hero-cinematic
height: 100dvh
pin: false
triggers:
  - target: ".hero-bg"
    scroll: "top top -> bottom top"
    scrub: true
    props: "scale 1 -> 1.15, yPercent 0 -> 18"
  - target: ".hero-title .line"
    onEnter: "stagger 0.08s, y 40 -> 0, opacity 0 -> 1, ease power3.out"
reduced_motion: "Static final frame; fade opacity 0 -> 1 over 0.3s only"
mobile_fallback: "Disable parallax yPercent; keep entrance fade"
```

## Core Architecture & Sync Loop

Never run separate `requestAnimationFrame` loops for Lenis, GSAP, and Three.js. Drive everything through `gsap.ticker`:

```js
import gsap from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
import Lenis from 'lenis';

gsap.registerPlugin(ScrollTrigger);
ScrollTrigger.config({ ignoreMobileResize: true }); // Prevents iOS URL-bar jumps

const lenis = new Lenis({ lerp: 0.1, smoothWheel: true });
lenis.on('scroll', ScrollTrigger.update);

gsap.ticker.add((time) => {
  lenis.raf(time * 1000);
});
gsap.ticker.lagSmoothing(0);

// Refresh triggers after web fonts & images settle
document.fonts.ready.then(() => ScrollTrigger.refresh());
window.addEventListener('load', () => ScrollTrigger.refresh());

// Route anchor links through Lenis so smooth scroll doesn't fight native jumps
document.querySelectorAll('a[href^="#"]').forEach((link) => {
  link.addEventListener('click', (e) => {
    const target = document.querySelector(link.getAttribute('href'));
    if (target && lenis) {
      e.preventDefault();
      lenis.scrollTo(target);
    }
  });
});
```

### Responsive & Reduced-Motion Gate (`gsap.matchMedia`)

Wrap all heavy scroll choreography inside `gsap.matchMedia()`. It auto-reverts timelines on breakpoint or preference changes:

```js
const mm = gsap.matchMedia();

mm.add({
  isDesktop: '(min-width: 1024px)',
  isMobile: '(max-width: 1023px)',
  reduceMotion: '(prefers-reduced-motion: reduce)',
}, (context) => {
  const { isDesktop, reduceMotion } = context.conditions;

  if (reduceMotion) {
    // Render content in final readable state; skip scrub/pin/parallax
    return;
  }

  if (isDesktop) {
    // Pinned storytelling, horizontal scroll, magnetic cursor, full 3D scrub
  } else {
    // Mobile: prefer vertical flow or CSS scroll-snap; lighter 3D DPR (<= 1.5)
  }
});
```

## The 11 Effect Modules (M01–M11)

See [references/motion-modules.md](references/motion-modules.md) for production code recipes and tunable parameters for each module:

| ID | Module | Primary Tech | Key Technique |
|---|---|---|---|
| **M01** | Cinematic Hero | GSAP Timeline + Scrub | Staggered title + slow bg scale + parallax |
| **M02** | Parallax Image | ScrollTrigger / CSS `view()` | Multi-layer `yPercent` at different speeds |
| **M03** | Text Reveal | GSAP + accessible spans | `aria-label` on parent, `aria-hidden` on split lines/chars |
| **M04** | Image Mask Transition | CSS `clip-path` + GSAP | `inset()` / `circle()` / SVG mask scrub |
| **M05** | Sticky Storytelling | `ScrollTrigger` `pin: true` | Fixed stage with crossfading chapters |
| **M06** | Horizontal Scroll | `ScrollTrigger` + `x` | `x: () => -(track.scrollWidth - innerWidth)` + `invalidateOnRefresh` |
| **M07** | 3D Camera Movement | Three.js + ScrollTrigger | Fixed `<canvas>`, DOM sections drive camera/object |
| **M08** | Magnetic Cursor | `gsap.quickTo()` | Desktop pointer-fine only (`(pointer: fine)`) |
| **M09** | Particle / Shader BG | Three.js + GLSL | Uniform `uProgress` / `uMouse` + viewport culling |
| **M10** | Morphing Transition | SVG / `clip-path` / WebGL | Seamless scene-to-scene visual bridge |
| **M11** | Pseudo-3D Sprite Stacking | 2D Canvas + Camera Lerp | Multi-slice volumetric objects + seamless Top/Chase/HUD views |

## Engineering & Accessibility Guardrails

- **Compositor-only properties:** Animate `transform`, `opacity`, `clip-path`, and `filter`—never `top`, `left`, `width`, `height`, or `margin`.
- **Viewport units:** Use `min-height: 100dvh` (or `svh`) instead of `100vh` to prevent mobile browser bar layout shifts.
- **Horizontal overflow:** Use `overflow-x: clip` (not `overflow-x: hidden`, which breaks `position: sticky`).
- **CSS Scroll-driven Gotcha:** Always declare `animation-timeline: view()` *after* the `animation` shorthand, and guard with `@supports (animation-timeline: scroll())`.
- **Three.js Hygiene:**
  - **Lazy-load WebGL:** Dynamically `import()` Three.js scenes only when `prefers-reduced-motion: no-preference` so reduced-motion users don't download ~450KB+ of unused 3D code.
  - Cap pixel ratio: `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))` (or `1.5` on mobile).
  - **Viewport culling:** Skip `renderer.render(scene, camera)` when the canvas is off-screen (via `IntersectionObserver` or `ScrollTrigger` `onToggle`).
  - **Cleanup:** Dispose geometries, materials, textures, and renderer on unmount/scene swap.
- **Breakpoint Parity:** Keep CSS `@media` breakpoints and `gsap.matchMedia()` conditions aligned to the exact pixel (e.g., CSS `max-width: 900px` ↔ JS `min-width: 901px`) so horizontal tracks never misalign.
- **Runtime Reduced-Motion Toggle:** Support an in-page reduced-motion toggle alongside OS `prefers-reduced-motion: reduce`; on activation, call `lenis.destroy()`, `mm.revert()`, and `gsap.set(targets, { clearProps: 'all' })`.
- **Accessible Split Text:** Never split text into `<span>`s without `aria-label` on the parent and `aria-hidden="true"` on the child spans, or screen readers will spell out individual characters.

## Browser Verification Checklist

Before declaring any section complete:
- [ ] `ScrollTrigger` `markers: true` checked in dev: `start`/`end` align cleanly; no overlapping pin-spacers.
- [ ] `document.fonts.ready` and `resize` keep pin positions accurate.
- [ ] DevTools Performance trace shows 60fps scrolling with no layout thrashing.
- [ ] Mobile viewport (`< 768px`) has zero horizontal overflow and usable touch scrolling.
- [ ] `prefers-reduced-motion: reduce` emulated in DevTools shows all content immediately readable with no jarring motion.
