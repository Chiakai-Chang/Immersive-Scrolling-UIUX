# Immersive Scrolling UI/UX Skill（沉浸式捲動介面設計技能）

[English](README.md) | [繁體中文（臺灣）](README.zh-TW.md)

一套專為 AI Coding Agent 打造的工程級 UI/UX 技能（Skill），協助 Agent 建置電影級的捲動敘事體驗，整合 **GSAP ScrollTrigger**、**Lenis**、**Three.js / React Three Fiber**、**2D Canvas 偽 3D** 與 **原生 CSS 捲動驅動動畫（`animation-timeline`）**。

> **核心理念：** 真正高級的沉浸式網頁，不是零散的特效堆砌，而是讓背景縮放、前景視差、遮罩揭露、字體動態與 3D 運鏡共用同一條時間軸與節奏。

---

## 專案結構

```text
.
├── README.md
├── README.zh-TW.md
└── skills/
    └── immersive-scrolling-uiux/
        ├── SKILL.md                          # 核心開發流程、Motion Spec 模板、單一 ticker 與工程護欄
        └── references/
            ├── motion-modules.md             # M01–M12 特效實作配方 + Web Motion Playground 模式
            ├── architecture-and-webgl.md     # React useGSAP、R3F 橋接、Shader 預熱、紋理圖集與效能分級
            └── accessibility-and-polish.md   # 3D 螢幕閱讀器雙生 DOM、毛玻璃降級、View Transitions 與狀態降級
```

---

## 安裝方式

### 給 AI Coding Agent 的自動安裝指令（可直接貼 Repo 網址請 Agent 執行）

如果你是 AI Coding Agent（Gemini CLI、Claude Code、Codex、Copilot CLI 或 Cursor），當使用者貼上本專案網址 `https://github.com/Chiakai-Chang/Immersive-Scrolling-UIUX` 並要求安裝時，請將 `skills/immersive-scrolling-uiux` 複製到使用者的跨平台技能目錄（`~/.agents/skills/immersive-scrolling-uiux`）並驗證 `SKILL.md` 存在：

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

### 手動安裝（從本機複製）

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

## 這套 Skill 強制落實的標準

1. **反樣板設計簡報與 Motion Spec 先行** — 動手寫碼前先鎖定頁面語法（Page Grammar）、1 個專屬招牌互動、情緒節拍弧線（Beat-Based Narrative Arc），以及每個區塊的 YAML 動態規格。
2. **逐段實作、逐段瀏覽器驗收** — 不一次生出整頁未驗證的動畫；每完成一段就檢查 `ScrollTrigger` 標記、確認無 Dead Scroll（無效捲動）與 Ghost Copy（殘留半透明文字）、動態背景對比度（`>= 4.5:1`）、60fps、手機觸控、鍵盤 `Tab` 導覽與 `prefers-reduced-motion`。
3. **單一渲染迴圈架構** — Lenis 與 Vanilla Three.js / 2D Canvas 統一由 `gsap.ticker` 驅動（`lagSmoothing(0)`），搭配 `ScrollTrigger.config({ ignoreMobileResize: true })`、`document.fonts.ready` 重整、Lenis 錨點導覽與 R3F `useFrame` 橋接。
4. **工程效能與無障礙護欄**：
   - 僅動畫 GPU 合成屬性、LCP 安全首屏（不對首屏 `<h1>` 設初始 `opacity: 0`）、使用 `min-height: 100dvh` 與 `overflow-x: clip`
   - `gsap.matchMedia()` 搭配 `AbortController` 清理原生事件監聽器、CSS 與 JS **斷點完全對齊**、React `useGSAP()` 生命週期清理
   - 橫向捲動區塊支援 WCAG 2.2 `focusin` 自動對齊、純 3D／畫布場景提供 `ScreenReaderOverlay` 語義雙生 DOM、Split Text 強制使用 `aria-label` + `aria-hidden`
   - 支援頁面內「減少動態」即時切換、`prefers-reduced-transparency` 毛玻璃降級、`HIGH / MEDIUM / LOW` 硬體效能分級、`gl.compileAsync()` 著色器預熱、紋理圖集（Texture Atlas）打包與資源 `dispose()` 清理

---

## 12 大特效模組（M01–M12）

完整可複製程式碼請見 [`skills/immersive-scrolling-uiux/references/motion-modules.md`](skills/immersive-scrolling-uiux/references/motion-modules.md)：

| 編號 | 模組名稱 | 主要技術 |
|---|---|---|
| **M01** | Cinematic Hero（電影級首屏） | GSAP Timeline + ScrollTrigger Scrub |
| **M02** | Parallax Image（視差滾動） | 原生 CSS `animation-timeline: view()` / GSAP `data-speed` |
| **M03** | Text Reveal（無障礙文字揭露） | GSAP + `aria-label` / `aria-hidden` 拆字結構 |
| **M04** | Image Mask Transition（遮罩轉場） | CSS `clip-path`（`inset` / `circle`）+ GSAP |
| **M05** | Sticky Storytelling（釘選敘事） | `ScrollTrigger` `pin: true` + 章節淡入淡出 |
| **M06** | Horizontal Scroll（橫向捲動） | `ScrollTrigger` `x` + `invalidateOnRefresh` + `containerAnimation` |
| **M07** | 3D Camera Movement（3D 鏡頭運鏡） | Three.js / R3F 固定 `<canvas>` + ScrollTrigger 路徑點 |
| **M08** | Magnetic & Diffused Cursor（磁吸與擴散游標） | `gsap.quickTo()`（僅限 `(pointer: fine)` 與純 `scale` 動畫） |
| **M09** | Particle / GLSL Shader BG（粒子與著色器背景） | Three.js `ShaderMaterial` / `Points` + 視口外暫停渲染 |
| **M10** | Morphing / Scene Bridge（變形與場景橋接） | SVG path / `clip-path` / CSS 主題色切換 |
| **M11** | Pseudo-3D Sprite Stacking（偽 3D 精靈堆疊） | 2D Canvas 多層切片 + 俯視／追逐／HUD 多視角切換 |
| **M12** | Frame-Sequence Scrub（影格序列捲動） | 2D Canvas 漸進載入序列圖 + ScrollTrigger `snap` |

---

## 參考資料與致謝（References & Inspiration）

### 官方文件與技術資源
- [GSAP ScrollTrigger Docs](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- [Lenis Smooth Scroll](https://lenis.darkroom.engineering/)
- [Three.js Documentation](https://threejs.org/)
- [MDN CSS Scroll-driven Animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Scroll-driven_animations)
- [Codrops](https://tympanus.net/codrops/)

### 研習與借鏡之開源專案
- [nateherkai/scroll-craft](https://github.com/nateherkai/scroll-craft) — 反樣板防撞版機制與 Dead Scroll / 動態對比自檢思維
- [adgoodma/interactive-web-designer](https://github.com/adgoodma/interactive-web-designer) — 毛玻璃動態背景疊加規範、完整互動狀態與節制的設計細節（Design Spells）
- [zlbigger/immersive-poetry-page](https://github.com/zlbigger/immersive-poetry-page) — 情緒節拍分段（Beat-Based Narrative Arc）與單一視覺聖經一致性
- [cortiz2894/showcase-images](https://github.com/cortiz2894/showcase-images) — `OffscreenCanvas` 紋理圖集打包與物理慣性虛擬捲動
- [Sunil56224972/The-Drawn-Dimenstion](https://github.com/Sunil56224972/The-Drawn-Dimenstion) — `gl.compileAsync()` 著色器預熱、硬體效能分級與 `ScreenReaderOverlay` 雙生無障礙層
- [Itssanthoshhere/Macbook-Landing-Page](https://github.com/Itssanthoshhere/Macbook-Landing-Page) 與 [udithavithanage/3d-web](https://github.com/udithavithanage/3d-web) — R3F + `@gsap/react` 實戰架構
- [rudra-xi/memoir-veritas](https://github.com/rudra-xi/memoir-veritas)、[Abdul-Ahad888/ScrollWorld](https://github.com/Abdul-Ahad888/ScrollWorld)、[zyna-b/React-GSAP-Animated-Studio-website](https://github.com/zyna-b/React-GSAP-Animated-Studio-website)、[giksaw/valentines2026](https://github.com/giksaw/valentines2026) — GSAP 視覺敘事、WebP 素材管線與工作室轉場互動
- [latentcat/latentbox](https://github.com/latentcat/latentbox) — Web3D 與創意程式設計精選合集（`awesome-web3d`），適合作為前期視覺研究靈感庫
- UI8 *Fly*（`remix.ui8.ai/fly`）— M11 超輕量（<32KB）2D Canvas 精靈堆疊與多視角 HUD 靈感來源
