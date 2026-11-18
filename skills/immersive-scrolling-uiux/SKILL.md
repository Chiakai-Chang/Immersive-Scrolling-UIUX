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
- Creating a **Web Motion Playground** or reusable motion module library (M01–M12).

**When NOT to use:** Standard SaaS dashboards, data tables, or CRUD forms where scroll-jacking or pinned sections hurt task efficiency. Pair with `frontend-design` or `design-taste-frontend` for static visual direction, and `motion-choreography` for standard UI state transitions.

## Technology Decision Matrix

Choose the lightest tool that satisfies the choreography (and if the project already uses GSAP, Motion/Framer Motion, or R3F, respect the existing stack before proposing new dependencies):

| Effect Requirement | Use | Why |
|---|---|---|
| Simple progress bar, card fade/scale on scroll | **CSS `animation-timeline: scroll()` / `view()`** | Zero JS, compositor-thread smooth, Baseline 2026 |
| Pinned sections, scrubbed timelines, horizontal scroll | **GSAP + ScrollTrigger** | Deterministic pin-spacers, `scrub`, `containerAnimation` |
| Smooth wheel inertia | **Lenis** (synced to `gsap.ticker`) | Decouples wheel smoothing without breaking `ScrollTrigger` |
| Lightweight 3D objects, multi-camera views, HUDs (<32KB) | **2D Canvas Pseudo-3D (Sprite Stacking)** | Zero WebGL bundle cost; instant FCP, volumetric look |
| Shader-only / 2D-plane WebGL effects (~30KB) | **OGL** (driven by ScrollTrigger) | Minimal WebGL footprint when 3D geometry isn't needed |
| Full PBR lighting, 3D `.glb` models, camera paths | **Three.js / R3F** (driven by ScrollTrigger) | Full 3D WebGL control; keep DOM as the scroll height driver |
| Pointer-following / magnetic elements | **`gsap.quickTo()`** | Reuses a single tween; never spawn new tweens per `mousemove` |

## Mandatory Workflow

Do **not** generate an entire multi-section site in one shot. Follow this 5-step loop:

1. **Design Brief (Anti-Template Gate):** Lock theme, palette, typography, a **Feeling Curve** (one emotion per section—**never repeat the same emotion or motion device twice in a row**), a **Page Grammar** with explicit *forbids* (so the layout doesn't drift back to a generic template), **1 bespoke signature mechanic**, and **dimensional hero planes** (independent foreground/subject/background separation). Use real copy or explicit `[Placeholder]` markers—never invent fake stats, heritage badges, or decorative AI-slop tells (see [references/accessibility-and-polish.md](references/accessibility-and-polish.md)).
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
    onEnter: "stagger 0.08s, yPercent 105 -> 0 (overflow: hidden mask, LCP-safe, no opacity: 0), ease power3.out"
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
// NEVER enable ScrollTrigger.normalizeScroll(true) alongside Lenis — they fight over wheel/touch events.

const lenis = new Lenis({ lerp: 0.1, smoothWheel: true, anchors: true });
lenis.on('scroll', (l) => {
  // Note: Lenis reports progress = 1 on non-scrollable pages; guard with l.limit > 0
  ScrollTrigger.update();
});

gsap.ticker.add((time) => {
  lenis.raf(time * 1000);
});
gsap.ticker.lagSmoothing(0);

// Refresh triggers after web fonts & images settle
document.fonts.ready.then(() => ScrollTrigger.refresh());
window.addEventListener('load', () => ScrollTrigger.refresh());
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

  // ALWAYS clean up native DOM listeners (mousemove, focusin) on breakpoint/toggle revert
  const ac = new AbortController();
  // target.addEventListener('mousemove', handler, { signal: ac.signal });
  return () => ac.abort();
});
```

## The 12 Effect Modules (M01–M12) & References

- **Effect Recipes (M01–M12 + Playground):** [references/motion-modules.md](references/motion-modules.md)
- **React / R3F, Shader Warmup, Texture Atlas & Performance Tiering:** [references/architecture-and-webgl.md](references/architecture-and-webgl.md)
- **Screen-Reader DOM Twin, Glass Overlays, View Transitions & Fallbacks:** [references/accessibility-and-polish.md](references/accessibility-and-polish.md)

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
| **M12** | Frame-Sequence Scrub | 2D Canvas + ScrollTrigger | Apple-style image sequence spin with poster fallback |

## Engineering & Accessibility Guardrails

- **Compositor-only properties:** Animate `transform`, `opacity`, `clip-path`, and `filter`—never `top`, `left`, `width`, `height`, or `margin`.
- **ScrollTrigger Official Rules:** Never animate the `pin: true` element itself—pin the wrapper and animate its children. Use `clamp(top bottom)` on above-the-fold scrubbed triggers, `ScrollTrigger.batch()` for card grids instead of N individual triggers, and `refreshPriority` if triggers are registered out of DOM order.
- **LCP-Safe Hero:** Never set initial `opacity: 0` via JS on the above-the-fold Hero `<h1>` or LCP hero image (delays First/Largest Contentful Paint). Animate subtle `y` transform or CSS-only entrance instead.
- **Framework / SPA Cleanup:** In React/Next.js, wrap all GSAP/ScrollTrigger code in `useGSAP(() => { ... }, { scope: containerRef })` from `@gsap/react` (or `gsap.context()` + `ctx.revert()` on unmount) to prevent StrictMode duplicate triggers and memory leaks.
- **Keyboard Accessibility (`focusin`):** For pinned horizontal sections (M06), listen to `focusin` on the track and scroll the window to keep the focused panel visible (WCAG 2.2 2.4.11/2.4.13); never hide focus rings.
- **Viewport units:** Use `min-height: 100dvh` (or `svh`) instead of `100vh` to prevent mobile browser bar layout shifts.
- **Horizontal overflow:** Use `overflow-x: clip` (not `overflow-x: hidden`, which breaks `position: sticky`).
- **CSS Scroll-driven Gotcha:** Always declare `animation-timeline: view()` *after* the `animation` shorthand, and guard with `@supports (animation-timeline: scroll())`.
- **Three.js & Canvas Hygiene:**
  - **Lazy-load WebGL:** Dynamically `import()` Three.js scenes only when `prefers-reduced-motion: no-preference` so reduced-motion users don't download ~450KB+ of unused 3D code. Wire ScrollTrigger camera tweens inside the `.then()` / `await` resolution (guarded against revert/unmount).
  - **Shader Warmup & Texture Atlas:** Call `gl.compileAsync(scene, camera)` during preloader/idle (bypass on low-tier mobile to prevent `webglcontextlost`), and pack multi-image WebGL galleries into a single `OffscreenCanvas` **Texture Atlas** (`1` draw call instead of `N`).
  - **Canvas Accessibility Twin:** For pure-3D or M11 Canvas experiences, provide a synchronized `.sr-only` semantic `<nav>` + `aria-live="polite"` region so screen readers and keyboard users can navigate every scene.
  - **Adaptive Performance Tiering (`HIGH / MEDIUM / LOW`):** Gate DPR (`2` / `1.5` / `1`), shadows, antialiasing, and particle density (`100%` / `60%` / `30%`) using `navigator.hardwareConcurrency`, `navigator.deviceMemory`, and a runtime FPS `downgradeTier()` fallback—not just viewport width.
  - Cap pixel ratio: `renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))` (or `1.5` on mobile).
  - **Viewport culling:** Skip both Three.js `renderer.render()` and 2D Canvas `ctx` draw calls when the canvas is off-screen (via `IntersectionObserver` or `ScrollTrigger` `onToggle`).
  - **Cleanup:** Dispose geometries, materials, textures, and renderer on unmount/scene swap.
- **Breakpoint Parity:** Keep CSS `@media` breakpoints and `gsap.matchMedia()` conditions aligned to the exact pixel (e.g., CSS `max-width: 900px` ↔ JS `min-width: 901px`) so horizontal tracks never misalign.
- **Glass / Overlay Hygiene:** When placing frosted/glass cards over moving WebGL, video, or scrubbed backdrops, cap stacked `backdrop-filter` blur layers at 3, keep worst-case text contrast `>= 4.5:1`, and honour `prefers-reduced-transparency`.
- **Complete Interactive States:** Explicitly define `hover`, `focus-visible`, `loading` (poster/skeleton while 3D or frame sequences decode), and `error`/fallback states—never ship only the happy path.
- **Restrained "Design Spells":** Keep celebratory micro-interactions optional, throttled, and dismissible; delight is seasoning, never a blocker to the core scroll/reading flow.
- **Runtime Reduced-Motion Toggle:** Support an in-page reduced-motion toggle alongside OS `prefers-reduced-motion: reduce`; on activation, call `lenis.destroy()`, `mm.revert()`, and `gsap.set(targets, { clearProps: 'all' })`.
- **Accessible Split Text:** Never split text into `<span>`s without `aria-label` on the parent and `aria-hidden="true"` on the child spans, or screen readers will spell out individual characters.

## Browser Verification Checklist

Before declaring any section complete:
- [ ] `ScrollTrigger` `markers: true` checked in dev: `start`/`end` align cleanly; no overlapping pin-spacers.
- [ ] **No Dead Scroll / Frozen Scrub / Ghost Copy:** Sample scrubbed sections on **entry, mid-pin, and exit** (never only mid-pin); verify fixed/canvas stages update a `data-verify-state` signature rather than sitting frozen, and revealed text reaches `opacity: 1` with `>= 4.5:1` contrast over dynamic backgrounds.
- [ ] `document.fonts.ready` and `resize` keep pin positions accurate.
- [ ] DevTools Performance trace shows 60fps scrolling with no layout thrashing.
- [ ] Mobile viewport (`< 768px`) has zero horizontal overflow and usable touch scrolling.
- [ ] Keyboard `Tab` navigation through pinned/horizontal sections keeps every focused element visible in viewport.
- [ ] `prefers-reduced-motion: reduce` emulated in DevTools shows all content immediately readable with no jarring motion.
