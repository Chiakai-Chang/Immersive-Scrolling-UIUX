# Immersive Scrolling UI/UX Skill

An AI coding agent skill for building cinematic, scroll-driven web experiences with **GSAP ScrollTrigger**, **Lenis**, **Three.js / WebGL**, and **CSS Scroll-driven Animations** (`animation-timeline`).

## What's Inside

- [`skills/immersive-scrolling-uiux/SKILL.md`](skills/immersive-scrolling-uiux/SKILL.md) — Core workflow, Motion Specification template, unified `gsap.ticker` architecture, `gsap.matchMedia()` responsive/accessibility gates, and browser verification checklist.
- [`skills/immersive-scrolling-uiux/references/motion-modules.md`](skills/immersive-scrolling-uiux/references/motion-modules.md) — Production code recipes for 10 core modules (**M01–M10**):
  - **M01** Cinematic Hero
  - **M02** Parallax Image (CSS `view()` & GSAP `data-speed`)
  - **M03** Accessible Text Reveal
  - **M04** Image Mask Transition (`clip-path`)
  - **M05** Sticky / Pinned Storytelling
  - **M06** Horizontal Scroll (`containerAnimation`)
  - **M07** Three.js 3D Camera & Object Scrub
  - **M08** Magnetic Cursor (`gsap.quickTo`)
  - **M09** Particle / GLSL Shader Background
  - **M10** Morphing / Scene Bridge Transitions

## Installation

Copy the skill folder into your cross-runtime agent skills directory (`~/.agents/skills/`):

```bash
# macOS / Linux
cp -r skills/immersive-scrolling-uiux ~/.agents/skills/

# Windows (PowerShell)
Copy-Item -Recurse -Force .\skills\immersive-scrolling-uiux "$env:USERPROFILE\.agents\skills\"
```
