# Immersive Scrolling UI/UX Skill

An AI coding agent skill for building cinematic, scroll-driven web experiences with **GSAP ScrollTrigger**, **Lenis**, **Three.js / WebGL**, and **CSS Scroll-driven Animations** (`animation-timeline`).

> **Core Principle:** High-end immersive websites are not isolated animations—they are **choreographed timelines** where background scale, foreground parallax, mask reveals, typography, and 3D camera motion share a unified rhythm.

---

## Repository Structure

```text
.
├── README.md
└── skills/
    └── immersive-scrolling-uiux/
        ├── SKILL.md                      # Core workflow, Motion Spec, ticker & a11y guardrails
        └── references/
            └── motion-modules.md         # Production recipes for M01–M10 + Playground pattern
```

---

## Installation

Copy `skills/immersive-scrolling-uiux` into your cross-runtime agent skills directory (`~/.agents/skills/`, recognized by Gemini CLI, Claude Code, Codex, and Copilot CLI):

### macOS / Linux
```bash
mkdir -p ~/.agents/skills
cp -r skills/immersive-scrolling-uiux ~/.agents/skills/
```

### Windows (PowerShell)
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills" | Out-Null
Copy-Item -Recurse -Force .\skills\immersive-scrolling-uiux "$env:USERPROFILE\.agents\skills\"
```

---

## What This Skill Enforces

1. **Motion Specification Before Code** — Agents write a per-section YAML spec (`start`/`end`, `scrub`, `pin`, `reduced_motion`, `mobile_fallback`) before implementing.
2. **Section-by-Section Browser Verification** — Never dump an entire multi-section site unverified; validate markers, 60fps compositor performance, mobile layout, and `prefers-reduced-motion` per section.
3. **Single Unified Render Loop** — Lenis and Three.js are driven through `gsap.ticker` with `gsap.ticker.lagSmoothing(0)`, `ScrollTrigger.config({ ignoreMobileResize: true })`, `document.fonts.ready` refresh, and Lenis anchor-link routing.
4. **Engineering & Accessibility Guardrails**:
   - Compositor-only animations (`transform`, `opacity`, `clip-path`, `filter`)
   - `min-height: 100dvh` & `overflow-x: clip` (preserving `position: sticky` / `pin`)
   - Exact **Breakpoint Parity** between CSS `@media` and `gsap.matchMedia()`
   - Accessible split text (`aria-label` on parent, `aria-hidden="true"` on child spans)
   - Runtime reduced-motion toggle support + Three.js lazy-loading, DPR cap, `IntersectionObserver` viewport culling, and `dispose()` cleanup

---

## The 10 Effect Modules (M01–M10)

See [`skills/immersive-scrolling-uiux/references/motion-modules.md`](skills/immersive-scrolling-uiux/references/motion-modules.md) for production code patterns:

| ID | Module | Primary Tech |
|---|---|---|
| **M01** | Cinematic Hero | GSAP Timeline + ScrollTrigger Scrub |
| **M02** | Parallax Image | CSS `animation-timeline: view()` / GSAP `data-speed` |
| **M03** | Text Reveal | GSAP + accessible `aria-label` / `aria-hidden` spans |
| **M04** | Image Mask Transition | CSS `clip-path` (`inset` / `circle`) + GSAP |
| **M05** | Sticky Storytelling | `ScrollTrigger` `pin: true` + chapter crossfade |
| **M06** | Horizontal Scroll | `ScrollTrigger` `x` + `invalidateOnRefresh` + `containerAnimation` |
| **M07** | 3D Camera Movement | Three.js fixed `<canvas>` + ScrollTrigger waypoints |
| **M08** | Magnetic Cursor | `gsap.quickTo()` (`(pointer: fine)` only) |
| **M09** | Particle / GLSL Shader BG | Three.js `ShaderMaterial` / `Points` + viewport culling |
| **M10** | Morphing / Scene Bridge | SVG path / `clip-path` / CSS theme variable transitions |

---

## References

- [GSAP ScrollTrigger Docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- [Lenis Smooth Scroll](https://lenis.darkroom.engineering/)
- [Three.js Documentation](https://threejs.org/)
- [MDN CSS Scroll-driven Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations)
- [Codrops](https://tympanus.net/codrops/)
