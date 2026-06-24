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
| `aurora.html` | Aurora borealis curtains | Planned |
| `galaxy.html` | Spiral galaxy gravity wells | Planned |
| `ocean.html` | Deep-ocean bioluminescence | Planned |
| `lava.html` | Flowing lava / domain warp | Planned |
| `gravity_sand.html` | GPU particle sand physics | Planned |
| `kaleidoscope.html` | Real-time mirror mandala | Planned |
| `sound_visualiser.html` | Microphone-driven visuals | Planned |
| `bubble_wrap.html` | Poppable bubble grid | Planned |
| `lightning.html` | Multi-touch lightning network | Planned |
| `shatter.html` | Voronoi glass shatter | Planned |

See `sensory_room_spec.md` for the full technical specification.

## Adding a new animation

1. Build the `.html` file (self-contained, no external dependencies).
2. In `index.html`, set the matching item's `file:` to the new filename so its tile goes live.
3. Commit and push — GitHub Pages updates automatically.
