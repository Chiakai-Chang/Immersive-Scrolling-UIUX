# Immersive Scrolling UI/UX Skill

[English](README.md) | [繁體中文（臺灣）](README.zh-TW.md)

An AI coding agent skill for building cinematic, scroll-driven web experiences with **GSAP ScrollTrigger**, **Lenis**, **Three.js / React Three Fiber**, **2D Canvas Pseudo-3D**, and **CSS Scroll-driven Animations** (`animation-timeline`).

> **Core Principle:** High-end immersive websites are not isolated animations—they are **choreographed timelines** where background scale, foreground parallax, mask reveals, typography, and 3D camera motion share a unified rhythm.

---

## Repository Structure

```text
.
├── README.md
├── README.zh-TW.md
└── skills/
    └── immersive-scrolling-uiux/
        ├── SKILL.md                          # Core workflow, Motion Spec, ticker & guardrails
        └── references/
            ├── motion-modules.md             # M01–M12 production recipes + Playground pattern
            ├── architecture-and-webgl.md     # React useGSAP, R3F bridge, Shader Warmup, Atlas & Tiering
            └── accessibility-and-polish.md   # ScreenReaderOverlay, Glass, View Transitions & Fallbacks
```

---

## Installation

### For AI Coding Agents (Self-Install Instructions)

If you are an AI coding agent (Gemini CLI, Claude Code, Codex, Copilot CLI, or Cursor) asked to install this skill from `https://github.com/Chiakai-Chang/Immersive-Scrolling-UIUX`, copy `skills/immersive-scrolling-uiux` into the user's cross-runtime skills directory (`~/.agents/skills/immersive-scrolling-uiux`) and verify `SKILL.md` exists:

```bash
# macOS / Linux
git clone --depth 1 https://github.com/Chiakai-Chang/Immersive-Scrolling-UIUX.git /tmp/immersive-scrolling-uiux && \
mkdir -p ~/.agents/skills && \
rm -rf ~/.agents/skills/immersive-scrolling-uiux && \
cp -r /tmp/immersive-scrolling-uiux/skills/immersive-scrolling-uiux ~/.agents/skills/ && \
rm -rf /tmp/immersive-scrolling-uiux && \
ls -la ~/.agents/skills/immersive-scrolling-uiux/SKILL.md
```

```powershell
# Windows (PowerShell)
$tmp = Join-Path $env:TEMP "immersive-scrolling-uiux"
if (Test-Path $tmp) { Remove-Item -Recurse -Force $tmp }
git clone --depth 1 https://github.com/Chiakai-Chang/Immersive-Scrolling-UIUX.git $tmp
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills" | Out-Null
Copy-Item -Recurse -Force "$tmp\skills\immersive-scrolling-uiux" "$env:USERPROFILE\.agents\skills\"
Remove-Item -Recurse -Force $tmp
Test-Path "$env:USERPROFILE\.agents\skills\immersive-scrolling-uiux\SKILL.md"
```

### Manual Installation (From Local Clone)

```bash
# macOS / Linux
mkdir -p ~/.agents/skills
cp -r skills/immersive-scrolling-uiux ~/.agents/skills/
```

```powershell
# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills" | Out-Null
Copy-Item -Recurse -Force .\skills\immersive-scrolling-uiux "$env:USERPROFILE\.agents\skills\"
```

---

## What This Skill Enforces

1. **Anti-Template Design Brief & Motion Spec** — Agents lock a distinct Page Grammar, 1 bespoke signature mechanic, a Beat-Based Narrative Arc, and a per-section YAML Motion Spec before coding.
2. **Section-by-Section Browser Verification** — Validate `ScrollTrigger` markers, zero Dead Scroll / Ghost Copy, dynamic contrast (`>= 4.5:1`), 60fps compositor performance, mobile layout, keyboard `Tab` focus, and `prefers-reduced-motion` per section.
3. **Single Unified Render Loop** — Lenis and Vanilla Three.js/2D Canvas are driven through `gsap.ticker` (`lagSmoothing(0)`), with `ScrollTrigger.config({ ignoreMobileResize: true })`, `document.fonts.ready` refresh, Lenis anchor routing, and R3F `useFrame` bridge support.
4. **Engineering & Accessibility Guardrails**:
   - Compositor-only animations, LCP-safe Hero (no initial `opacity: 0` on `<h1>`), `min-height: 100dvh`, and `overflow-x: clip`
   - `gsap.matchMedia()` with `AbortController` listener cleanup, exact CSS/JS **Breakpoint Parity**, and React `useGSAP()` lifecycle patterns
   - WCAG 2.2 `focusin` scroll alignment for horizontal sections, `ScreenReaderOverlay` DOM twin for 3D/Canvas worlds, and accessible split text (`aria-label` + `aria-hidden`)
   - Runtime reduced-motion toggle, `prefers-reduced-transparency` glass fallback, adaptive `HIGH/MEDIUM/LOW` hardware tiering, `gl.compileAsync()` shader warmup, Texture Atlas packing, and `dispose()` cleanup

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
| **M07** | 3D Camera Movement | Three.js / R3F fixed `<canvas>` + ScrollTrigger waypoints |
| **M08** | Magnetic & Diffused Cursor | `gsap.quickTo()` (`(pointer: fine)` only, scale-only) |
| **M09** | Particle / GLSL Shader BG | Three.js `ShaderMaterial` / `Points` + viewport culling |
| **M10** | Morphing / Scene Bridge | SVG path / `clip-path` / CSS theme variable transitions |
| **M11** | Pseudo-3D Sprite Stacking | 2D Canvas multi-slice + Top/Chase/HUD camera lerp |
| **M12** | Frame-Sequence Scrub | 2D Canvas image sequence + ScrollTrigger `snap` |

---

## References & Inspiration

### Core Documentation
- [GSAP ScrollTrigger Docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- [Lenis Smooth Scroll](https://lenis.darkroom.engineering/)
- [Three.js Documentation](https://threejs.org/)
- [MDN CSS Scroll-driven Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations)
- [Codrops](https://tympanus.net/codrops/)

### Open-Source Skills & Projects Studied
- [greensock/gsap-skills](https://github.com/greensock/gsap-skills) — Official GSAP AI skills (pin-wrapper separation, `clamp()`, `ScrollTrigger.batch()`, `refreshPriority` & `gsap.quickTo()`)
- [pbakaus/impeccable](https://github.com/pbakaus/impeccable) — Child-surface hover stability (no edge flicker), visitor-mode pacing & default-visible fallback discipline
- [juliobsz/skills](https://github.com/juliobsz/skills) — `awwwards` / `jury` quality bar, OGL lightweight WebGL, `@media (hover: hover)` parity, CSS letter-roll & parallel page transitions
- [nateherkai/scroll-craft](https://github.com/nateherkai/scroll-craft) — Feeling Curve, Page Grammar *forbids*, dimensional hero planes & entry/exit scrub verification
- [AThevon/genjutsu](https://github.com/AThevon/genjutsu) — Anti-AI-Slop tells catalogue, existing-stack respect & interaction thesis discipline
- [saralobo/rules-design-bible](https://github.com/saralobo/rules-design-bible) — M3 asymmetric enter/exit easing, <100ms feedback & 100th-repetition micro-interaction discipline
- [rastian/interaction-design-skills](https://github.com/rastian/interaction-design-skills) — Nielsen response-time loading thresholds (`100ms / 1s / 10s`) & IxD affordance/feedback principles
- [adgoodma/interactive-web-designer](https://github.com/adgoodma/interactive-web-designer) — Glass overlay hygiene, complete interactive states & restrained design-spell discipline
- [zlbigger/immersive-poetry-page](https://github.com/zlbigger/immersive-poetry-page) — Beat-based narrative arc & single visual bible consistency
- [cortiz2894/showcase-images](https://github.com/cortiz2894/showcase-images) — `OffscreenCanvas` Texture Atlas packing & physics-based virtual scroll
- [Sunil56224972/The-Drawn-Dimenstion](https://github.com/Sunil56224972/The-Drawn-Dimenstion) — `gl.compileAsync()` shader warmup, adaptive hardware performance tiering & `ScreenReaderOverlay`
- [Itssanthoshhere/Macbook-Landing-Page](https://github.com/Itssanthoshhere/Macbook-Landing-Page) & [udithavithanage/3d-web](https://github.com/udithavithanage/3d-web) — R3F + `@gsap/react` showcase patterns
- [rudra-xi/memoir-veritas](https://github.com/rudra-xi/memoir-veritas), [Abdul-Ahad888/ScrollWorld](https://github.com/Abdul-Ahad888/ScrollWorld), [zyna-b/React-GSAP-Animated-Studio-website](https://github.com/zyna-b/React-GSAP-Animated-Studio-website), [giksaw/valentines2026](https://github.com/giksaw/valentines2026) — GSAP storytelling, WebP asset pipelines & studio UI transitions
- [latentcat/latentbox](https://github.com/latentcat/latentbox) — Curated Web3D & creative-coding showcase directory (`awesome-web3d`) for visual research
- UI8 *Fly* (`remix.ui8.ai/fly`) — Inspiration for M11 lightweight (<32KB) 2D Canvas Sprite Stacking & multi-camera HUD UX
