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
- **`fluid_sensory.html`** — GPU Navier–Stokes fluid. Complete and tuned. Refactored onto the framework.
- **`template.html`** — canonical framework + a minimal Canvas2D placeholder animation.
- **`sync-framework.js`** — `node sync-framework.js` copies the `FRAMEWORK-*` regions from
  `template.html` into every other `.html` (ANIMATION blocks untouched). Run it after any framework edit.
- **`index.html`** — landing page; tiles auto-generated from an `ITEMS` array (`file:null` = "Soon").
- **`sensory_room_spec.md`** — trimmed spec (7 sections). Aurora/Galaxy/Ocean/Lava were removed from the plan.

### Architecture (important)
Every animation = **FRAMEWORK** (shared, identical, marked `FRAMEWORK-CSS/HTML/JS START…END`)
+ **ANIMATION** block defining one `Anim` object:
```js
const Anim = { sectionLabel, themes, defaults, schema,
  init(canvas), resize(w,h), splat(x,y,dx,dy,color,opts), frame(dt), reset() };
```
- `splat` coords are normalised 0..1, **y points up**.
- Framework already provides: settings + localStorage, colour system (themes + swatches + lock +
  background), full audio engine (9 voices, 4 scales, reverb/echo/chorus, master limiter),
  5-touch input with pointer pruning, the ≡ menu → Appearance/Sound tabs panel, DPR sizing,
  WebGL context-loss recovery, the render loop (calls `Anim.splat` per touch sample, then `Anim.frame(dt)`).
- Edit framework ONLY in `template.html`, then `node sync-framework.js`.

## NEXT TASK: build `flock.html` (abstract steering swarm)
Boids (separation/alignment/cohesion + wander + per-finger seek/flee). **CPU + Canvas2D first**
(spatial-hash neighbours), GPU version possible later.

Locked design decisions:
- **Per-theme forms** — each theme changes palette + motion + agent shape:
  - Murmuration — cool blue/white, fast/tight, **velocity streaks**
  - Fireflies — warm amber/green, slow/loose/pulsing, **glow dots**
  - Plasma — magenta/cyan neon, medium/spiky, **constellation** (neighbour lines)
  - Embers — orange/red, upward drift + flicker, **sparks**
  - Aurora — green/violet, very slow, **soft blobs** + long trails
- **Touch:** attract by default + **Repel toggle**.
- **Colour-claim:** agents tint toward the colour of the finger they follow; drift back to theme when released.
- **Trails:** soft fade, with a length slider.
- **Settings schema:** Agent count (~500 default, 100–1500) · Speed · Cohesion · Separation ·
  Alignment · Touch pull · Trail length · Glow size · **Repel** toggle.
- Audio: reuse the framework's per-touch tonal engine as-is.

### Framework upgrade required for flock
The Appearance schema currently renders **sliders only**. Add support for **toggle items**
(`{type:'toggle', key, label}`) in `buildPanel`/the schema loop so flock's Repel toggle can live
there. Do it in `template.html`, keep backward-compatible (fluid has only sliders), then sync.

### Rollout checklist for flock
1. Copy `template.html` → `flock.html`, replace ANIMATION block.
2. Implement boids + spatial hash + per-theme render styles + colour-claim + trails.
3. Add framework toggle-schema support (template.html) + `node sync-framework.js`.
4. `index.html`: set the Flock tile's `file:` to `flock.html` (tile already added, currently Soon).
5. Syntax-check, test in preview (5 touches, themes, repel, reset), commit + push.

## Remaining roadmap (after flock)
gravity_sand · kaleidoscope · sound_visualiser · bubble_wrap · lightning · shatter.

## Candidate animation ideas (CS algorithms — not committed, no build order)
Captured brainstorm. Most are GPU ping-pong shader sims like fluid_sensory (reuse that pattern:
splat() seeds the field, frame() steps + renders, themes = colour map + rule preset). Slime mould
is an agent system like flock.

Strong recommends:
- **Reaction–Diffusion (Gray–Scott)** — two diffusing "chemicals" → organic spots/stripes/coral
  (Turing patterns) that grow from touch. Calm, hypnotic. GPU.
- **Lenia / SmoothLife** — continuous (floating-point) Game of Life: soft lifelike organisms that
  glide/pulse/divide; touch seeds life. The "beautiful Game of Life." GPU. (Classic binary GoL is a
  weak sensory fit — flickery, dies out; if ever done, use slow tick + colour-by-age + touch reseed.)
- **Slime mould (Physarum)** — thousands of agents lay/follow trails → glowing branching networks
  that rewire around your touch. Showstopper, alive when idle. Agent sim (GPU or CPU).

Also good:
- **Cyclic cellular automata** — self-organising spiral waves; touch seeds spirals. GPU.
- **Wave / ripple sim (2D wave equation)** — still pond; touches send interfering ripples. Very
  calming, instant cause-and-effect, great with 5 fingers. GPU/cheap.
- **Diffusion-Limited Aggregation (DLA)** — crystals/coral grow outward from each touch point.
- **Cymatics / Chladni patterns** — sound frequency → standing-wave patterns; pairs with the audio
  engine (touch height = frequency). Cross-over with the sound side.
- **Flow-field particles (Perlin noise)** — calm particle streams along a field touch warps. Sibling
  of flock — probably pick one, not both.

## DEFERRED idea: alternative input devices (accessibility) — framework-level
Discussed, not started. **Do this after the animations exist** (framework enhancement → every
animation benefits at once; sits alongside/just before the lighting layer). Audience: students with
physical + learning disabilities. Principle: support a few standard browser inputs, since most
accessibility hardware emulates keyboard/mouse/gamepad.

Add input adapters that feed the EXISTING input pipeline (onDown/onMove/onUp → Anim.splat); no
per-animation work needed. Priority order:
1. **Keyboard + switches** (biggest win, tiny effort). Big-button switches/switch interfaces emulate
   keys (space/enter) or mouse clicks. Add a **scanning / auto-move mode**: a cursor/attractor drifts
   automatically and a switch press fires an effect at its position → true single-switch access.
2. **Gamepad API** — Xbox controller AND the **Xbox Adaptive Controller** (XAC presents as a standard
   gamepad + has 3.5mm jacks for external switches/joysticks, so this covers a lot). Stick → virtual
   pointer/attractor; buttons → taps/bursts. Easy, no deps.
3. **Webcam motion** (getUserMedia + cheap frame-difference) — contactless; motion centroids become
   virtual pointers. NOTE: avoid ML pose/hand tracking (needs a model file → breaks offline/no-deps).
4. **Microphone** — already covered by the planned `sound_visualiser`.
5. **Web MIDI** — optional; adaptive music devices / big-pad controllers (SEN music rooms).
6. Eye-gaze / accessibility joysticks usually emulate a mouse pointer + dwell-click → covered by mouse
   support + keeping targets large/forgiving.

All browser-native, dependency-free, offline-friendly. Implement as framework input adapters so
keyboard/switch/gamepad/webcam all flow through the same pointer pipeline.

## DEFERRED idea: room lighting that mirrors on-screen colour (DMX / WLED / Philips Hue)
Discussed, not started. **Do this only AFTER all the touch animations are built and working** —
it's the final layer, not to be interleaved with the animation work. Goal: room lights match the
dominant on-screen colour.

Key constraint: a sandboxed browser page can't drive DMX/UDP and is blocked (mixed-content,
self-signed cert, CORS) from reaching LAN lights directly. So:
- **WLED** — easy; takes RGB. JSON API (throttle ~5–10/s) or UDP realtime DRGB (smooth, needs a relay).
- **Philips Hue** — needs a local relay: CLIP v2 API is HTTPS self-signed + app key + ~10 cmds/s
  (too slow/awkward for the browser); the smooth path is the **Entertainment API over DTLS/UDP ~50Hz**,
  impossible from a page. Hue wants CIE **xy + brightness** (RGB→xy with per-lamp gamut clamp).
- **DMX** — USB-serial (Enttec) via Web Serial, or Art-Net/sACN over UDP (relay).

**Recommended architecture:** a small **`bridge/` Node app on the room PC** that serves the pages
over http://localhost (avoids mixed-content/CORS) and fans one "current colour" out to WLED (UDP),
Hue (Entertainment/DTLS + RGB→xy), and optionally DMX. Plus a tiny **optional framework hook** that
emits a representative colour ~15Hz (average of active touch colours, fading to theme/dim when idle;
or 1×1 canvas downsample). If no bridge is configured/reachable it silently no-ops, so pages stay
standalone and still open from USB.

One-time setup later: WLED device IPs/segments; Hue link-button app key + an Entertainment area.

Honest caveat: this feature inherently needs the bridge (or Web Serial hardware) — the animations
stay self-contained; lighting is an opt-in layer on top.

## Conventions
- Commit messages end with the Co-Authored-By trailer used in existing commits.
- `git push` after each logical change is fine (user has approved pushing).
- CRLF/LF warnings on commit are harmless (Windows).
- Hardware target: Intel Iris Xe / modern integrated GPU @ 1080p = 60fps. Fluid `DYE_RES` is the
  perf lever if a weak machine struggles.
