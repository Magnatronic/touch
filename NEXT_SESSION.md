# Resume notes — Touch sensory animations

Working notes for picking up in a fresh CLI session. Delete or trim once stale.

## What this project is
A suite of standalone, offline-capable, fullscreen **multi-touch sensory-room animations**
for a touch projector. Each is a **single self-contained `.html`** (no build, no deps, no
network) that also runs offline from `file://`. Hosted on GitHub Pages.

- Repo: https://github.com/Magnatronic/touch  (owner: Magnatronic, branch `main`)
- Live: https://magnatronic.github.io/touch/
- Local: `C:\Local Docs\Sensory\Interactive`

## Current state (done)
- **`fluid_sensory.html`** — GPU Navier–Stokes fluid. Complete. 6 themes (✦Cosmic/🔥Lava/🌊Ocean/🌌Aurora/⚡Neon/🌈Prism). 4 Style presets (Ink/Smoke/Storm/Silk).
- **`slime.html`** — GPU Physarum slime mould. Complete. 6 themes (🍄Mycelium/⚡Neon/🌕Gold/🌊Biolum/🔥Ember/🌈Prism). 4 Style presets (Gossamer/Dense/Rivers/Chaos).
- **`flock.html`** — CPU boids swarm. Complete. 6 colour-only themes (🩵Sky/🌿Garden/💜Violet/🔥Embers/🌌Aurora/🌈Prism). 5 Style presets (Murmuration/Fireflies/Plasma/Embers/Aurora) that set physics + renderStyle via SETTINGS.
- **`life.html`** — Conway's Game of Life. Complete. 6 themes (💎Crystal Cave/🪸Coral Reef/🔥Ember Forge/🕸️Gossamer/🟩Neon Grid/🌈Prism). 4 Style presets (Classic/Microscope/Pixel/Neon).
- **`template.html`** — canonical framework + minimal placeholder animation.
- **`sync-framework.js`** — copies FRAMEWORK-* regions from template.html into every other .html. Run after any framework edit.
- **`index.html`** — landing page with Live/Soon tiles.

### Architecture (important)
Every animation = **FRAMEWORK** (shared, identical, marked `FRAMEWORK-CSS/HTML/JS START…END`)
+ **ANIMATION** block defining one `Anim` object:
```js
const Anim = { sectionLabel, themes, defaults, schema,
  init(canvas), resize(w,h), splat(x,y,dx,dy,color,opts), frame(dt), reset() };
```
- `splat` coords are normalised 0..1, **y points up**.
- Framework provides: settings + localStorage, colour system (themes + swatches + lock + background), full audio engine (9 voices, 4 scales, reverb/echo/chorus, master limiter), 5-touch input with pointer pruning, the ≡ menu panel, DPR sizing, WebGL context-loss recovery, render loop.
- Edit framework ONLY in `template.html`, then `node sync-framework.js`.

### New UI (as of this session)
≡ opens a two-row bar:
- **Quick row** (top): context-sensitive chips — Theme chips / Style chips / Colour row (↻ Auto + 8 fixed swatches + 🎨 picker + bg:🎨)
- **Bottom row**: 🎨 Theme · ✨ Style · 💛 Colour · 🎛️ Controls · 🔊 Sound · 🧹
- Controls panel: animation sliders + performance + Reset settings
- Sound panel: unchanged
- Style button hidden if `Anim.styles` is absent
- `Anim.styles` contract: `{ key: { label, ...SETTINGS_keys } }` — tapping a chip merges values into SETTINGS

---

## Remaining animation roadmap
gravity_sand · kaleidoscope · sound_visualiser · bubble_wrap · lightning · shatter

## Candidate animation ideas (not committed)
- Reaction–Diffusion (Gray–Scott) — GPU, Turing patterns
- Lenia / SmoothLife — continuous Game of Life, GPU
- Cyclic cellular automata — self-organising spirals, GPU
- Wave / ripple sim — 2D wave equation, GPU, very calming
- Diffusion-Limited Aggregation (DLA) — crystals from touch
- Flow-field particles (Perlin noise) — calm streams

## Deferred: accessibility input adapters
Keyboard/switch scanning mode · Gamepad (Xbox Adaptive) · Webcam motion · Web MIDI.
Framework-level, add after animations are built.

## Deferred: room lighting (DMX / WLED / Hue)
Needs a local bridge Node app. Do after all animations exist. See previous NEXT_SESSION.md for full architecture.

## Conventions
- Commit messages end with the Co-Authored-By trailer used in existing commits.
- `git push` after each logical change is fine.
- CRLF/LF warnings on commit are harmless (Windows).
- Hardware target: Intel Iris Xe / modern integrated GPU @ 1080p = 60fps.
- Syntax check: extract JS to temp file first — `node --check` fails on .html extension in Node v24.
