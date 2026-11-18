# Accessibility, Glass Overlays & State Completeness

Load this reference when building **3D/Canvas navigation**, **frosted/liquid glass overlays** over dynamic backdrops, **View Transitions**, or **loading/fallback states**.

---

## 1. Screen-Reader DOM Twin for 3D / Canvas Worlds (`ScreenReaderOverlay`)

When interactive scenes live inside `<canvas>` (M07, M09, M11), maintain a synchronized `.sr-only` semantic `<nav>` and an `aria-live="polite"` region so screen readers and keyboard users can navigate every room/chapter:

```html
<div class="sr-overlay" role="complementary" aria-label="Accessible scene navigation">
  <nav class="sr-only" aria-label="Chapters">
    <button type="button" data-goto="0">Chapter 1: Overview</button>
    <button type="button" data-goto="1">Chapter 2: Detail View</button>
  </nav>
  <div id="scene-live" aria-live="polite" aria-atomic="true" class="sr-only"></div>
</div>
```

Update `#scene-live` text whenever the active 3D/Canvas chapter changes.

---

## 2. Frosted / Liquid Glass over Dynamic Backdrops

Cap stacked `backdrop-filter` blur layers at 3, verify `>= 4.5:1` contrast against the worst-case background frame, and honour `prefers-reduced-transparency`:

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

---

## 3. View Transitions API (Seamless State / Filter Morph)

```js
export function transitionState(updateDom) {
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if (!document.startViewTransition || reduced) {
    updateDom();
    return;
  }
  document.startViewTransition(updateDom);
}
```

---

## 4. Loading, Preloader & Error Fallback States

- **Non-Blocking Preloader:** Animate progress via `transform: scaleX()` (never `width`), and never block the above-the-fold Hero `<h1>` LCP behind artificial `setTimeout` delays.
- **3D / Sequence Status:** Render a visible `[data-stage-status="loading"]` poster/skeleton until assets decode, and switch to `[data-stage-status="fallback"]` (static hero image) on `webglcontextlost` or asset load failure.
