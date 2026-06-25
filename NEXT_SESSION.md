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
