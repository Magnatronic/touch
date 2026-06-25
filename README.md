# Touch

Fullscreen, multi-touch sensory-room animations for a touch projector. Each animation is a
**single self-contained HTML file** — no build step, no dependencies, no network requests.
They run fullscreen, respond to up to five simultaneous fingers, synthesise all audio with the
Web Audio API, and work both online (GitHub Pages) and offline (opened straight from a USB stick).

## Live site

https://magnatronic.github.io/touch/

## Running offline

Download or clone the repo and open `index.html` (or any animation's `.html` file) directly in
Chrome — everything works with no internet connection.

## Animations

| File | Effect | Status |
|---|---|---|
| `fluid_sensory.html` | Liquid Navier–Stokes fluid simulation | ✅ Built |
| `gravity_sand.html` | GPU particle sand physics | Planned |
| `kaleidoscope.html` | Real-time mirror mandala | Planned |
| `sound_visualiser.html` | Microphone-driven visuals | Planned |
| `bubble_wrap.html` | Poppable bubble grid | Planned |
| `lightning.html` | Multi-touch lightning network | Planned |
| `shatter.html` | Voronoi glass shatter | Planned |

See `sensory_room_spec.md` for the full technical specification.

## Architecture: framework + animation

Every animation file is split into two clearly-marked regions:

- **FRAMEWORK** — shared scaffolding, identical in every file: settings + `localStorage`
  persistence, the colour system, the audio engine (voices, scales, effects, limiter),
  5-touch input, the settings panel, and the lifecycle/render loop. Marked with
  `FRAMEWORK-CSS / -HTML / -JS START … END` comments. **Do not hand-edit per file.**
- **ANIMATION** — the only part that differs per effect (`ANIMATION START … END`). It defines
  one `Anim` object the framework drives:

  ```js
  const Anim = {
    sectionLabel,           // heading for the animation's sliders
    themes,                 // { key: { label, colors:[[r,g,b]×5] } }
    defaults,               // animation-specific settings (merged into shared defaults)
    schema,                 // [{key,label,min,max,step,dp}] sliders
    init(canvas){},         // create rendering context + resources (re-run on GPU context loss)
    resize(w,h){},          // reallocate size-dependent buffers
    splat(x,y,dx,dy,color,opts){},  // inject one touch sample (x,y are 0..1, y up)
    frame(dt){},            // render one frame
    reset(){},              // clear the canvas / state
  };
  ```

`template.html` is the canonical framework with a minimal placeholder animation — copy it to
start a new effect, then replace only the ANIMATION block.

## Keeping the framework in sync

Edit the framework **once** in `template.html`, then propagate it to every animation:

```
node sync-framework.js
```

It copies each `FRAMEWORK-*` region from `template.html` into all other `.html` files,
leaving each ANIMATION block untouched. Runs at dev time only — the shipped files stay
standalone.

## Adding a new animation

1. Copy `template.html` to e.g. `aurora.html`.
2. Replace the ANIMATION block with your effect (keep the `Anim` contract).
3. In `index.html`, set the matching item's `file:` to the new filename so its tile goes live.
4. Commit and push — GitHub Pages updates automatically.
