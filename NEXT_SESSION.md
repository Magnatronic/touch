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
- **`fluid_sensory.html`** — GPU Navier–Stokes fluid. Complete and tuned.
- **`slime.html`** — GPU Physarum slime mould. Complete and tuned. Has 4 Style presets (Gossamer/Dense/Rivers/Chaos).
- **`flock.html`** — CPU boids swarm. Complete. 5 themes (Murmuration/Fireflies/Plasma/Embers/Aurora). NOTE: flock themes currently bundle colour + physics + render style together — needs splitting into colour-only themes + Style presets in a future pass (see UI redesign below).
- **`life.html`** — Conway's Game of Life. Complete. 6 themes, touch painting, birth notes, smooth fade transitions, 3 colour modes (Touch/Position/Mixed — Mixed now correctly inherits parent colours then drifts to position colour over ~80 generations).
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

---

## NEXT TASK: Framework UI redesign + Style presets + Prism theme

This is a framework-level change — edit `template.html`, run `node sync-framework.js`,
then add animation-specific `styles` and `prism` theme to each animation's ANIMATION block.

### 1. New UI interaction model

**Current:** ≡ → [✨ Appearance] [🔊 Sound] → scrolling panel

**New:** ≡ → two rows:
- **Quick row** (top): context-sensitive chips that change based on which bottom button is active
- **Bottom row**: [🎨 Theme] [✨ Style] [🎨 Colour] [🎛️ Controls] [🔊 Sound] [🧹]

```
[idle, faded]     ≡

[≡ tapped]
  💎 Crystal  🪸 Coral  🔥 Ember ...         ← quick row (Theme mode by default)
  [Theme] [Style] [Colour] [Controls] [Sound] [🧹]
```

### 2. Quick row states (context-sensitive)

| Bottom button tapped | Quick row shows | Panel opens? |
|---|---|---|
| **Theme** (default) | Theme chips for current animation | No |
| **Style** | Style preset chips for current animation | No |
| **Colour** | ↻ Auto · colour swatches · 🎨 picker · ┊ · bg 🎨 | No |
| **Controls** | (nothing in quick row) | Yes — Controls panel |
| **Sound** | (nothing in quick row) | Yes — Sound panel |
| **🧹** | — | No — direct clear action |

- Tapping the active bottom button again deactivates it (collapses quick row / closes panel).
- Active button is visually highlighted.
- 🧹 is always a direct action — never opens a panel.

### 3. Controls panel contents (replaces Appearance panel)
All existing settings preserved — just reorganised:
- Section header = Anim.sectionLabel
- All Anim.schema items (sliders, toggles, preset-chips) — unchanged
- Performance chips (Auto/High/Medium/Low)
- Show performance stats toggle
- ↺ Reset settings button (moved from panel footer to here)

### 4. Sound panel contents (unchanged)
Voice chips · Scale chips · Reverb/Echo/Chorus/Mute toggles · Volume slider

### 5. Colour quick row detail
```
↻  🔴  🟠  🟡  🟢  🔵  🟣  ⚪  🎨  ┊  bg:🎨
```
- ↻ Auto = cycle theme palette (existing behaviour)
- 8 standard colour swatches: red, orange, yellow, green, cyan, blue, purple, white
- 🎨 = touch colour custom picker (existing)
- ┊ = thin divider
- bg:🎨 = background colour picker (single picker, no presets — background rarely changes)
- Selecting any swatch locks that colour (existing lockColour behaviour)

### 6. Framework changes needed (template.html)

**New Anim contract field (optional):**
```js
styles: {
  ink:   { label: 'Ink',   /* keys matching SETTINGS keys and their preset values */ },
  smoke: { label: 'Smoke', ... },
}
```
When Style is tapped in the bottom row, quick row shows chips built from `Anim.styles`.
Tapping a style chip merges its values into SETTINGS and calls saveSettings().
If `Anim.styles` is absent, Style button is hidden/disabled.

**CSS additions needed:**
- Two-row bar layout when open
- Active state for bottom row buttons
- Standard colour swatch row styling

**JS additions needed:**
- `currentQuickMode` state ('theme'|'style'|'colour'|null)
- `showQuickMode(mode)` — renders correct quick row content
- Style chip builder from `Anim.styles`
- Standard colour row builder (8 fixed colours + custom picker + bg picker)
- Theme quick row builder (same chips as current themeRow but in the bar)
- Wire bottom row buttons to showQuickMode()
- 🧹 direct action wired in bar (not panel footer)

**Remove from framework:**
- The `#panel` Appearance pane (themeRow, colourGroup, appSliders, bgRow move out of panel)
- The ✨ Appearance cat button
- panelBtns div (Clear canvas moves to bar; Reset moves to Controls panel)

**Keep in framework:**
- Sound panel — entirely unchanged
- All existing JS functions: buildThemeButtons, buildSwatches, refreshColourUI, lockColour,
  buildBgRow, makeSlider, makeToggle, makeChips, pickColor, etc.

### 7. Style presets per animation

#### fluid_sensory.html
Add to Anim object:
```js
styles: {
  ink:   { label:'Ink',   brushSize:0.08, intensity:2000,  persistence:0.85 },
  smoke: { label:'Smoke', brushSize:0.12, intensity:5000,  persistence:0.55 },
  storm: { label:'Storm', brushSize:0.22, intensity:12000, persistence:0.20 },
  silk:  { label:'Silk',  brushSize:0.10, intensity:3000,  persistence:0.75 },
}
```

#### slime.html
Slime already has `preset` in its schema (Gossamer/Dense/Rivers/Chaos) wired to its own
preset system. Wire these to the new `styles` field instead (or in addition):
```js
styles: {
  gossamer: { label:'Gossamer', preset:'gossamer' },
  dense:    { label:'Dense',    preset:'dense' },
  rivers:   { label:'Rivers',   preset:'rivers' },
  chaos:    { label:'Chaos',    preset:'chaos' },
}
```
Keep the `preset` schema item in Controls for backwards compat.

#### flock.html (also fix themes/style separation)
Flock currently bundles colour + physics + render style inside each theme. Fix this:
- Split into colour-only themes (hue ranges, sat, lit, palette) and Style presets (physics + render).
- Style presets map to the old theme behaviour:
```js
styles: {
  murmuration: { label:'Murmuration', speed:1.8, cohesion:1.44, separation:1.04, alignment:2.08, trailLen:20, glowSize:4, renderStyle:'streaks' },
  fireflies:   { label:'Fireflies',   speed:0.55, cohesion:0.32, separation:0.64, alignment:0.52, trailLen:40, glowSize:7, renderStyle:'glow' },
  plasma:      { label:'Plasma',      speed:1.2, cohesion:0.52, separation:0.80, alignment:0.84, trailLen:15, glowSize:5, renderStyle:'constellation' },
  embers:      { label:'Embers',      speed:0.95, cohesion:0.24, separation:0.40, alignment:0.56, trailLen:8,  glowSize:3, renderStyle:'sparks' },
  aurora:      { label:'Aurora',      speed:0.25, cohesion:0.36, separation:0.44, alignment:0.32, trailLen:60, glowSize:9, renderStyle:'blobs' },
}
```
Themes become colour-only (palette + hue range). The existing render style switching
(streaks/glow/constellation/sparks/blobs) should be driven by SETTINGS.renderStyle
which Style presets set, rather than being hardcoded to the theme key.

#### life.html
```js
styles: {
  classic:    { label:'Classic',    speed:3,  cellSize:20, cellShape:'square', trail:3,  brush:'single' },
  microscope: { label:'Microscope', speed:5,  cellSize:8,  cellShape:'circle', trail:2,  brush:'single' },
  pixel:      { label:'Pixel',      speed:2,  cellSize:36, cellShape:'square', trail:1,  brush:'single' },
  neon:       { label:'Neon',       speed:8,  cellSize:14, cellShape:'dot',    trail:8,  brush:'single' },
}
```

### 8. Prism theme for all animations
Add a `prism` theme to every animation that doesn't already have one. Life already has Prism.

**Prism theme spec** (full-spectrum rainbow, works across all animations):
```js
prism: {
  label: '🌈 Prism',
  colors: [[1,0,0.2],[1,0.6,0],[0.2,1,0.2],[0,0.8,1],[0.7,0.2,1]],
  // animation-specific colour params vary per animation
}
```
- **fluid**: colors as above, no extra params needed
- **slime**: hueMin:0, hueMax:1.0, lMax:0.75, sat:1.0
- **flock**: hueMin:0, hueMax:1.0, sat:1.0, lit:0.62, drift:0 (use murmuration physics initially)
- **life**: ageHue0:0, ageHue1:360, ageSat:100, ageLit:58, ageLitRange:20, glow:false (already exists)
- **template**: add as example

### 9. Implementation order
1. `template.html` — new bar/quick row UI (framework JS + CSS + HTML)
2. `node sync-framework.js` — propagate to all animations
3. Add `styles` to each animation's Anim object (fluid, slime, flock, life)
4. Fix flock theme/style separation (colour-only themes + Style presets drive renderStyle)
5. Add `prism` theme to fluid, slime, flock (life already has it)
6. Verify all animations: themes switch, styles switch, colour picker works, controls unchanged, sound unchanged, clear works, reset works
7. Syntax-check each .html, test in browser, commit + push

### 10. What is NOT changing
- All existing schema settings (sliders, toggles, chips) remain in Controls panel — nothing removed
- Sound panel entirely unchanged
- localStorage persistence — all keys the same, styles just write to existing SETTINGS keys
- The Anim contract (init/resize/splat/frame/reset) — unchanged
- Performance system — unchanged, moves to Controls panel
- 5-touch input — unchanged

---

## Remaining animation roadmap (after UI redesign)
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
