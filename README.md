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
            └── motion-modules.md         # Production recipes for M01–M12, R3F, Glass & Playground
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

1. **Anti-Template Design Brief & Motion Spec** — Agents lock a distinct Page Grammar, 1 bespoke signature mechanic, and a per-section YAML Motion Spec before coding.
2. **Section-by-Section Browser Verification** — Validate `ScrollTrigger` markers, zero Dead Scroll / Ghost Copy, dynamic contrast (`>= 4.5:1`), 60fps compositor performance, mobile layout, keyboard `Tab` focus, and `prefers-reduced-motion` per section.
3. **Single Unified Render Loop** — Lenis and Vanilla Three.js/2D Canvas are driven through `gsap.ticker` (`lagSmoothing(0)`), with `ScrollTrigger.config({ ignoreMobileResize: true })`, `document.fonts.ready` refresh, Lenis anchor routing, and R3F `useFrame` bridge support.
4. **Engineering & Accessibility Guardrails**:
   - Compositor-only animations, LCP-safe Hero (no initial `opacity: 0` on `<h1>`), `min-height: 100dvh`, and `overflow-x: clip`
   - `gsap.matchMedia()` with `AbortController` listener cleanup, exact CSS/JS **Breakpoint Parity**, and React `useGSAP()` lifecycle patterns
   - WCAG 2.2 `focusin` scroll alignment for horizontal sections and accessible split text (`aria-label` + `aria-hidden`)
   - Runtime reduced-motion toggle, `prefers-reduced-transparency` glass fallback, Three.js lazy-loading, DPR cap, viewport culling, and `dispose()` cleanup

---

## The 12 Effect Modules (M01–M12)

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
| **M11** | Pseudo-3D Sprite Stacking | 2D Canvas multi-slice + Top/Chase/HUD camera lerp |
| **M12** | Frame-Sequence Scrub | 2D Canvas image sequence + ScrollTrigger `snap` |

---

## References

- [GSAP ScrollTrigger Docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- [Lenis Smooth Scroll](https://lenis.darkroom.engineering/)
- [Three.js Documentation](https://threejs.org/)
- [MDN CSS Scroll-driven Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations)
- [Codrops](https://tympanus.net/codrops/)
