# Sensory Room Interactive Animations — Technical Specification

## Project Overview

A suite of 11 standalone, offline-capable, fullscreen browser animations for a touch
projector in a college sensory room. Students include those with complex communication
needs, physical disabilities, and cognitive differences. The animations must be visually
stunning, highly responsive to multi-touch, and require no setup or technical knowledge
to operate.

Each animation is a **single self-contained HTML file**. There are no shared files,
no build steps, no frameworks, no package managers. Every file works when opened
directly in Chrome from a USB stick or local folder.

---

## Global Technical Requirements

### Rendering
- WebGL (GLSL fragment shaders) for all visual effects that benefit from GPU acceleration.
  Canvas 2D only where WebGL offers no advantage (e.g. simple 2D geometry without shader effects).
- 60fps target on a mid-range integrated GPU (Intel Iris Xe or equivalent).
- `devicePixelRatio`-aware canvas sizing — sharp on HiDPI displays.
- Responsive to any viewport — designed for 16:9 landscape but must not break at other ratios.

### Dependencies
- **Zero external dependencies.** No CDN links, no `<script src="...">`, no `import` from URLs.
- No network requests of any kind at runtime.
- No build tools, no npm, no bundlers.
- Every file is a single `.html` with all CSS, JavaScript, and GLSL inline.

### Touch & Input
- `touch-action: none` on the canvas element.
- All touch event listeners registered with `{ passive: false }`.
- Supports exactly **5 simultaneous independent touch points**.
- Each touch point must behave independently — different colours, sounds, and physics per finger.
- Mouse fallback for desktop testing (single pointer only).
- Touch coordinates must be correctly offset using `canvas.getBoundingClientRect()`.

### Audio
- All sound synthesised entirely with the **Web Audio API** — no audio files, no samples.
- Audio context must be created and resumed on the first user gesture (browser security
  requirement). Do not call `audioCtx.resume()` or create oscillators before a touch/click.
- Master `GainNode` at the output chain — volume controllable via a settings slider.
- Each simultaneous touch gets its own independent audio node chain.
- Stereo spatialisation via `StereoPanner` — horizontal touch position maps to pan position.
- Vertical touch position modulates pitch, filter cutoff, or timbre.
- All tones use pentatonic or modal scales so multiple simultaneous touches always harmonise.
- Graceful fade-in on touch start (`linearRampToValueAtTime`, 0.3–1.5s depending on animation).
- Graceful fade-out on touch end (`setTargetAtTime`, 0.3–2.0s depending on animation).

### Performance
- Particle simulations with >10,000 particles must run on the GPU (ping-pong render textures).
- Fragment shaders must avoid dynamic loop lengths where possible.
- FBM (fractional Brownian motion) noise octave counts must be configurable so performance
  can be tuned on slower hardware.
- No JavaScript in the render hot path that could cause GC pauses (pre-allocate typed arrays).

---

## Global UI Requirements

Every animation has the same two UI layers, overlaid on the canvas.

### Control Bar
- Position: fixed, bottom: 18px, horizontally centred.
- Contains: theme/mode buttons + a ⚙ Settings button.
- Opacity: 0.45 at rest; transitions to 1.0 on hover or when settings panel is open.
- Buttons styled as pills:
  - Default: `background: rgba(255,255,255,0.08)`, `border: 1px solid rgba(255,255,255,0.18)`,
    `border-radius: 22px`, `padding: 9px 18px`, `font-size: 13px`, `color: rgba(255,255,255,0.9)`,
    `font-family: sans-serif`
  - Active/hover: `background: rgba(255,255,255,0.22)`, `border-color: rgba(255,255,255,0.5)`
- All buttons must have `touch-action: manipulation` and `-webkit-tap-highlight-color: transparent`.
- Button click handlers must call `event.stopPropagation()` to prevent triggering the canvas.

### Settings Panel
- Triggered by the ⚙ button; slides up from the bottom of the screen.
- Transition: `transform: translateY(100%)` → `translateY(0)`, `transition: 0.3s ease`.
- Style: `background: rgba(0,0,0,0.82)`, `backdrop-filter: blur(14px)`,
  `border-radius: 16px 16px 0 0`, `padding: 20px 24px`, max-width: 500px, centred.
- Close button (✕) in top-right corner of the panel.
- A ↺ Reset button at the bottom of every settings panel — restores animation to initial state.
- All settings applied **live** as sliders are moved — no confirm step.
- Settings persisted to `localStorage` keyed by filename — survive page refresh.
- On load, read from `localStorage` and apply saved values before first render.

### Settings Control Styling
- Each control: a row with `display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 14px`.
- Label on left in `rgba(255,255,255,0.7)`, 13px.
- Current value on right in `rgba(255,255,255,0.5)`, 13px, updates live as slider moves.
- Slider (`<input type="range">`): full width, `margin-top: 6px`.
  Custom style: white thumb, dark transparent track, height 4px, border-radius 2px.
- Toggle: checkbox styled as a pill switch — label on left, switch on right.
- Section headers (if >6 settings): `font-size: 11px`, `letter-spacing: 0.1em`,
  `text-transform: uppercase`, `color: rgba(255,255,255,0.35)`, `margin: 16px 0 8px`.

---

## Animation Specifications

---

### 1. `fluid_sensory.html` — Fluid Simulation

**Sensory quality:** Visceral, liquid, flowing. High stimulation. Every touch leaves a lasting
trace that evolves long after the finger lifts.

**Visual concept:** Real-time Navier-Stokes fluid dynamics running entirely on the GPU.
Touch injects velocity and coloured dye into a fluid field. The fluid swirls, mixes, diffuses,
and dissipates. Multiple touches create competing vortices and colour mixing.

**Rendering — WebGL2 ping-pong framebuffers:**

Simulation runs on two pairs of framebuffers (velocity and dye), each 256×256 and 1024×1024
respectively. Pipeline each frame:

1. **Curl shader** — compute curl of velocity field
2. **Vorticity confinement shader** — amplify existing vortices to counteract numerical diffusion
3. **Divergence shader** — compute divergence of velocity
4. **Pressure clear** — multiply pressure buffer by `PRESSURE` constant
5. **Jacobi pressure solve** — iterate N times to solve Poisson equation
6. **Gradient subtract** — subtract pressure gradient from velocity to enforce incompressibility
7. **Advect velocity** — semi-Lagrangian advection with velocity dissipation
8. **Advect dye** — semi-Lagrangian advection with density dissipation
9. **Display** — render dye buffer to screen with additive blending and bloom

Each touch splat: inject velocity (dx, dy scaled by `SPLAT_FORCE`) and dye (theme colour)
into both buffers using a Gaussian splat shader (radius = `SPLAT_RADIUS / aspectRatio`).

Auto-splats fire every ~2.5 seconds when no touch is active to keep the fluid alive.

**Touch interaction:**
- Touch and drag: injects velocity in drag direction + coloured dye at touch point
- Each active touch gets a distinct colour from the current theme palette, cycling on each new touch
- 5 simultaneous touches each inject independently into the shared fluid field

**Themes:** Cosmic · Lava · Ocean · Aurora · Neon

Each theme defines 5 dye colours as RGB triples.

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Curl / Vorticity | 0 – 50 | 28 | Higher = tighter, more dramatic swirls |
| Pressure | 0 – 1 | 0.85 | How much pressure is retained each frame |
| Velocity dissipation | 0 – 0.5 | 0.18 | How quickly velocity fades |
| Density dissipation | 0.7 – 1.0 | 0.95 | How slowly dye fades (1.0 = never) |
| Splat radius | 0.05 – 0.8 | 0.22 | Size of each touch splat |
| Splat force | 1000 – 15000 | 7000 | Velocity injected per touch |
| Pressure iterations | 5 – 40 | 25 | Solver accuracy vs performance |
| Auto-splats | Toggle | On | Idle splats when no touch active |
| Volume | 0 – 1 | 0.30 | Master audio level |

**Audio:**
- Per-touch: sine oscillator, pitch from pentatonic scale based on x position,
  pitch modulated by y position and drag speed. LFO for gentle tremor.
- Stereo pan follows x. Fade-in 0.5s, fade-out 0.8s.

---

### 2. `aurora.html` — Aurora Borealis

**Sensory quality:** Calm, ethereal, slow. Low-medium stimulation. Works beautifully with
no touch at all — suitable for students who just need a calming visual environment.

**Visual concept:** Curtains of coloured light hang from the top of the screen, shifting
slowly with layered FBM (fractional Brownian motion) noise. Stars twinkle in darker areas.
Touch pulls the curtains toward the finger and creates a bright vertical pillar of light.

**Rendering — WebGL fragment shader (fullscreen quad):**

Each pixel's colour is computed per-frame in the fragment shader:

1. Compute 4 aurora bands. Each band has a horizontal position modulated by FBM noise over
   time, a vertical extent defined by a smoothstep envelope, and brightness modulated by
   a second FBM layer.
2. For each active touch: compute exponential falloff from pixel to touch coordinate
   (in aspect-corrected space); add a bright pillar (exponential falloff on x only) and
   warp the aurora sampling coordinates toward the touch point.
3. Render background star field: per-pixel hash function → point stars with sine-wave twinkle.
4. Colour the aurora using theme palette: mix between two colours based on FBM output,
   blend in a third colour at high brightness.
5. Apply a depth-sky gradient (dark at top, slight horizon glow at bottom).
6. Single-pass bloom approximation: multiply colour by luminance.
7. Subtle radial vignette.

**Touch interaction:**
- Touch: pulls aurora curtains toward finger; glowing pillar appears at touch x position
- Moving touch: pillar and warp follow continuously
- 5 independent touches each create their own pillar and warp region

**Themes:** Arctic (green/teal) · Sakura (pink/purple) · Solar (orange/gold) · Storm (blue/violet)

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Animation speed | 0.02 – 0.5 | 0.18 | How fast curtains drift |
| Curtain count | 1 – 6 | 4 | Number of layered aurora bands |
| FBM octaves | 2 – 8 | 6 | Curtain texture detail |
| Curtain reach | 0.2 – 0.95 | 0.55 | How far down the screen curtains hang |
| Touch pull strength | 0 – 3.0 | 1.8 | How strongly touch warps the curtains |
| Pillar brightness | 0 – 2.0 | 1.0 | Brightness of touch-point light pillar |
| Star density | 0 – 1 | 0.5 | Background star density |
| Star twinkle speed | 0 – 5 | 2.0 | How fast stars twinkle |
| Bloom | 0 – 1.5 | 0.5 | Glow around bright areas |
| Volume | 0 – 1 | 0.25 | Master audio level |

**Audio:**
- Per-touch: sine oscillator + octave-up sine at 0.3× gain (slight shimmer).
  Pitch from pentatonic scale, y position modulates filter cutoff.
  Fade-in 1.2s, fade-out 1.5s.
- No idle audio.

---

### 3. `galaxy.html` — Galaxy Gravity Wells

**Sensory quality:** Cosmic, dramatic, mesmerising. Medium-high stimulation.
Touch creates an immediate and satisfying visual explosion.

**Visual concept:** A spiral galaxy computed analytically in a fragment shader. Each active
touch point acts as a gravity well, warping the spiral arms toward the finger. Touching
creates a supernova flash. Stars twinkle independently.

**Rendering — WebGL fragment shader:**

Per pixel:
1. Compute polar coordinates (r, θ) from the aspect-corrected screen centre.
2. Accumulate gravitational pull from all active touch uniforms: for each touch,
   compute force = mass / (distance² + ε) directed toward the touch point.
   Sum forces and warp the sampling position.
3. Recompute polar coords from warped position.
4. Spiral arms: map `fract((θ / 2π) × armCount − r × tightness + time × speed)` → band
   via smoothstep. Multiply by radial falloff from centre.
5. Nebula: FBM noise layered over spiral arms.
6. Stars: hash-based point stars with sine twinkle.
7. Supernova: at each active touch, `exp(−d² × coreWidth)` bright core +
   `exp(−d² × haloWidth)` wide halo.
8. Colour using theme palette, bloom, vignette.

**Touch interaction:**
- Touch: bends spiral arms toward finger; supernova flash at touch position
- Moving touch: gravity well follows continuously; arms bend dynamically
- 5 independent gravity wells each warp the galaxy simultaneously

**Themes:** Deep Space · Nebula Rose · Solar · Vortex

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Rotation speed | 0 – 0.3 | 0.12 | Galaxy rotation rate |
| Spiral arm count | 1 – 6 | 3 | Number of spiral arms |
| Spiral tightness | 0.5 – 5.0 | 2.5 | How tightly arms wind inward |
| Gravity strength | 0 – 0.4 | 0.15 | How strongly touch warps the galaxy |
| Nebula density | 0 – 1 | 0.6 | Amount of nebula cloud texture |
| Star density | 0 – 1 | 0.5 | Star field density |
| Supernova size | 20 – 300 | 120 | Radius of touch flash effect |
| Core brightness | 0 – 3 | 1.4 | Brightness of galaxy centre |
| Bloom | 0 – 1 | 0.4 | Glow around bright areas |
| Volume | 0 – 1 | 0.28 | Master audio level |

**Audio:**
- Per-touch: fundamental sine + perfect 5th + octave (harmonic stack).
  Pitch from pentatonic scale, y modulates pitch. Fade-in 1.0s, fade-out 2.0s.

---

### 4. `ocean.html` — Deep Ocean

**Sensory quality:** Calm, deep, mysterious. Low-medium stimulation. Soothing for students
who are overwhelmed — the slow pace and low-frequency audio are grounding.

**Visual concept:** A bioluminescent deep-ocean environment. Jellyfish drift upward and pulse.
Plankton particles glow and drift. Caustic light patterns (light refracted through water
surface) shift across the scene. Touch creates pressure wave rings radiating outward and
triggers a bioluminescent burst.

**Rendering — WebGL fragment shader:**

Per pixel:
1. Sky-to-depth colour gradient (theme deep → mid colour, top to bottom).
2. Caustics: FBM noise sampled with slow time offset, powered to sharpen.
   Modulated by vertical position (stronger near top, like light from surface above).
3. Jellyfish (N instances): each is an SDF. Bell: ellipse SDF with scale pulsing at
   `0.7 + 0.3 × sin(time × rate + instanceIndex)`. Tentacles: 8 point SDFs below the
   bell, positions animated with sine functions. Accumulate glow from each.
4. Plankton: 30 point-light instances at pseudorandom drifting positions, each pulsing
   independently.
5. Touch pressure waves: for each active touch, accumulate 4–8 expanding rings using
   `exp(−(distance − ringRadius)² × sharpness) × sin(phase)` where ringRadius grows
   over time since touch.
6. Touch bioluminescent burst: radial glow at each touch position.
7. Bloom, vignette, depth fog at bottom.

**Touch interaction:**
- Touch: expanding pressure wave rings + bioluminescent burst at touch point
- Moving touch: wave origin follows finger; continuous ripples
- 5 independent touches each have their own wave system

**Themes:** Abyss (teal) · Coral (orange/red) · Algae (green) · Midnight (purple)

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Jellyfish count | 0 – 12 | 5 | Number of jellyfish |
| Jellyfish speed | 0 – 0.1 | 0.04 | Upward drift speed |
| Pulse rate | 0.5 – 4.0 | 1.8 | Bell pulse frequency |
| Plankton count | 0 – 60 | 30 | Bioluminescent particle count |
| Wave ring count | 1 – 8 | 4 | Rings per touch |
| Wave speed | 0.05 – 0.3 | 0.12 | Ring expansion speed |
| Caustic intensity | 0 – 2 | 0.8 | Brightness of caustic light patterns |
| Bio-glow brightness | 0 – 2 | 1.2 | Brightness of bioluminescent elements |
| Depth fog | 0 – 1 | 0.5 | Murk/depth effect at bottom |
| Volume | 0 – 1 | 0.25 | Master audio level |

**Audio:**
- Per-touch: sine oscillator through bandpass filter. Base frequency 55–110Hz
  (deep whale-like tone). Y position modulates filter cutoff (200–1200Hz).
  Fade-in 1.5s, fade-out 2.0s.

---

### 5. `lava.html` — Lava / Domain Warp

**Sensory quality:** Hypnotic, organic, primal. Medium stimulation. Beautiful even
without touch — the lava flows continuously.

**Visual concept:** Continuously flowing lava-like blobs rendered with double FBM domain
warping — the same technique used in classic ShaderToy lava lamp effects. Touch warps
and agitates the flow locally, creating bright veins of heat at contact points.

**Rendering — WebGL fragment shader:**

Per pixel:
1. Compute `q = vec2(fbm(st + t), fbm(st + vec2(offset) + t × 0.8))`.
2. Compute `r = vec2(fbm(st + q × warpStrength + vec2(...) + t × 1.1), fbm(...))`.
3. Compute `f = fbm(st + r × 2.0)`.
4. For each active touch: compute displacement from pixel to touch, apply a local
   warp `st += fbm(...) × turbulence × exp(−distance × falloff)`, and add a bright
   vein `col += themeHighlight × exp(−distance² × veinhardness)`.
5. Map `f` through a 4-stop colour gradient (theme c0 → c1 → c2 → c3) using `mix` and `pow`.
6. Bloom, vignette.

**Touch interaction:**
- Touch: warps flow locally, bright vein at contact point
- Moving touch: stirs and agitates — faster drag = more turbulence
- 5 independent touches each warp independently

**Themes:** Magma · Cryo (ice blue) · Toxic (acid green) · Bubblegum (pink)

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Flow speed | 0.02 – 0.5 | 0.15 | How fast the lava moves |
| Warp strength | 0.5 – 4.0 | 2.2 | Intensity of domain warp |
| FBM octaves | 2 – 8 | 6 | Texture detail |
| Touch warp radius | 0.5 – 5.0 | 2.5 | How far touch disturbance spreads |
| Touch warp strength | 0.05 – 0.4 | 0.18 | How strongly touch warps the flow |
| Vein brightness | 0.5 – 3.0 | 1.5 | Brightness of touch contact veins |
| Bloom | 0 – 1.5 | 0.6 | Glow around bright areas |
| Colour contrast | 0.5 – 3.0 | 1.0 | Steepness of colour gradient mapping |
| Volume | 0 – 1 | 0.25 | Master audio level |

**Audio:**
- Per-touch: sawtooth oscillator through waveshaper distortion (soft clip curve).
  Pitch from x position. LFO modulates pitch for organic tremor.
  Fade-in 0.8s, fade-out 1.5s.

---

### 6. `gravity_sand.html` — Gravity Sand

**Sensory quality:** Tactile, physical, satisfying. Medium stimulation. The sensation of
moving sand is deeply calming for many students. The gravity-tilt mechanic is dramatic
and exciting.

**Visual concept:** Tens of thousands of sand or snow grains that obey gravity, pile up
at the bottom, and respond to touch as directional wind forces. Holding multiple fingers
tilts the apparent gravity direction, causing avalanches.

**Rendering — WebGL GPU particle simulation:**

Two pairs of ping-pong RGBA float textures store particle state:
- Texture A: particle position (RG = xy, normalised 0–1)
- Texture B: particle velocity (RG = vx, vy)

Each frame:
1. **Physics update shader**: reads position and velocity textures, applies gravity vector,
   touch wind forces, damping, and boundary collisions. Writes new position and velocity.
2. **Display shader**: for each screen pixel, accumulate contributions from nearby particles
   using a spatial hash or by rendering particles as point sprites via `gl.POINTS`.
   Colour each particle by velocity magnitude (slow = base colour, fast = highlight).

Particle count configurable 20,000–100,000 (default 60,000).

**Physics model:**
- Gravity: constant downward acceleration (configurable strength).
- Wind (single touch): directed force away from touch point, magnitude = `strength × exp(−distance / radius)`.
- Vortex (two fingers close together): rotational force centred between the two fingers.
- Gravity tilt (3+ fingers held): gravity direction shifts toward the centroid of all active
  touch points. Returns to downward over ~2 seconds after all touches lift.
- Floor/wall collision: velocity component inverted × restitution factor.
- Damping: velocity multiplied by damping constant each frame.

**Themes:** Desert (ochre/gold on dark brown) · Snow (white/pale blue on dark blue) · Embers (orange/red, additive blending) · Neon (saturated colours on black, additive blending)

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Particle count | 20000 – 100000 | 60000 | Number of grains (reset required) |
| Gravity strength | 0.001 – 0.02 | 0.006 | Downward acceleration per frame |
| Damping | 0.90 – 0.999 | 0.97 | Velocity retained per frame |
| Wind strength | 0.001 – 0.05 | 0.015 | Force applied by touch |
| Wind radius | 0.05 – 0.5 | 0.20 | Radius of touch wind influence |
| Gravity return speed | 0.5 – 5.0 | 2.0 | How fast gravity returns after tilt |
| Particle size | 1 – 4 | 1.5 | Rendered size of each grain (px) |
| Restitution | 0 – 0.5 | 0.10 | Bounciness on floor/wall collision |
| Volume | 0 – 1 | 0.25 | Master audio level |

**Audio:**
- Touch drag: bandpass-filtered white noise (wind sound). Filter cutoff follows drag speed.
- Avalanche: second noise layer with amplitude following mean particle speed across the field.
- No pitched tones — textural only.

---

### 7. `kaleidoscope.html` — Kaleidoscope

**Sensory quality:** Creative, meditative, rewarding. Medium stimulation. Slow deliberate
movement creates beautiful mandalas; fast movement creates chaos. Ideal for students
who benefit from creative expression.

**Visual concept:** Touch draws glowing trails that are reflected across N symmetry axes
simultaneously, building real-time mandalas. The canvas rotates slowly when idle.
Double-tap or Reset clears the canvas.

**Rendering — WebGL render-to-texture:**

1. **Trail render pass**: each frame, render active touch trails into a wedge framebuffer
   (represents 1/N of a full circle, where N = symmetry count). Each trail segment is
   a Gaussian point (bright core + soft glow) rendered with additive blending.
   Apply a fade (multiply by `decayRate`) to the wedge buffer each frame.
2. **Display pass**: for each screen pixel, compute its polar coordinates (r, θ).
   Fold θ into [0, 2π/N] with mirroring at segment boundaries (fold even segments).
   Sample the wedge framebuffer at the folded position. Rotate the whole pattern slowly
   over time by offsetting θ before folding.

Touch trail colour cycles slowly over time (hue rotation), starting from a theme-defined
base hue.

**Interaction:**
- Touch and drag: paints glowing trails into the symmetry wedge
- Each touch has its own slowly-shifting hue
- No touch: existing pattern rotates slowly; soft idle colour pulse
- Double-tap OR Reset button: clear the wedge framebuffer

**Symmetry (theme buttons):** 3-fold · 6-fold · 8-fold · 12-fold

**Colour themes:** Rainbow · Gold · Neon · Pastel

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Symmetry | 2 – 16 | 6 | Mirror segments (overrides theme button) |
| Brush size | 2 – 40 | 12 | Radius of painted brush (pixels) |
| Glow radius | 5 – 80 | 24 | Soft glow radius around brush core |
| Trail brightness | 0.1 – 2.0 | 1.0 | Brightness of painted strokes |
| Trail decay | 0.950 – 0.9995 | 0.985 | How slowly trails fade |
| Rotation speed | 0 – 0.02 | 0.003 | Idle pattern rotation speed |
| Colour shift speed | 0 – 1.0 | 0.20 | How fast trail hue cycles |
| Background darkness | 0 – 1 | 0.95 | 0 = white background, 1 = black |
| Volume | 0 – 1 | 0.20 | Master audio level |

**Audio:**
- Per-touch: pure sine tone. Pitch mapped to distance from centre (further = higher),
  quantised to pentatonic scale. All simultaneous tones form a chord.
- Persistent sub-bass drone (55Hz, gain 0.05) always on once audio is unlocked.
- Touch tones fade over 1.5s when lifted.

---

### 8. `sound_visualiser.html` — Sound Visualiser

**Sensory quality:** Reactive, surprising, communal. The room itself becomes the instrument.
Clapping, music, voices, or instruments all drive the display. No touch required.

**Visual concept:** Microphone input drives the visuals in real time via Web Audio API FFT
analysis. Four distinct visual modes. Touch modifies colour and orientation.

**Input pipeline:**
```
getUserMedia({ audio: true })
  → MediaStreamAudioSourceNode
  → AnalyserNode (FFT)
  → Float32Array (frequency data, updated per frame)
  → 1D WebGL texture (uploaded per frame as uniform or texture)
  → Fragment shader
```

**Permission UX:**
- On load: black screen with a large centred prompt ("Tap anywhere to enable microphone").
  No permission request until first tap (avoids unexpected browser prompts).
- If permission denied: display a friendly message ("Microphone access needed — check browser
  settings") and fall back to a touch-driven idle pulse mode so the page is not broken.

**Visual modes (theme buttons):**

1. **Waveform** — The time-domain audio waveform is extruded vertically from the horizontal
   centre of the screen. Low frequencies are larger and glow warm; high frequencies are
   smaller and glow cool. The shape is mirrored top/bottom.

2. **Spectrum rings** — Frequency bands mapped to concentric rings radiating from centre.
   Bass frequencies = large outer ring. Treble = small inner rings. Rings pulse outward on
   transients. The whole pattern rotates slowly.

3. **Particle burst** — FFT energy drives a particle system. Loud broad-spectrum sound =
   particles spray outward from centre. Quiet = particles drift inward and orbit.
   Bass frequencies control particle size; treble controls colour.

4. **Fluid ripple** — Sound amplitude at each frequency band drives a corresponding ring
   of ripples across the screen. Louder = bigger ripple amplitude. Multiple frequency
   peaks create overlapping interference patterns.

**Touch interaction (all modes):**
- Tap anywhere: cycle to next colour palette
- Drag: rotate the visual (waveform and ring modes)
- 2+ fingers: scale the visual up/down

**Audio output:** None. This animation only listens. No audio output to prevent
microphone feedback.

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Sensitivity | 0.1 – 5.0 | 1.0 | FFT gain multiplier |
| FFT size | 256/512/1024/2048 | 1024 | Frequency resolution (stepped) |
| Smoothing | 0 – 0.95 | 0.80 | AnalyserNode smoothingTimeConstant |
| Bass boost | 0 – 3 | 1.0 | Multiplier for low-frequency bands |
| Treble boost | 0 – 3 | 1.0 | Multiplier for high-frequency bands |
| Particle count | 500 – 5000 | 2000 | Particles in burst mode |
| Rotation speed | 0 – 2 | 0.5 | Idle rotation speed |
| Bloom | 0 – 1.5 | 0.6 | Glow around bright elements |
| Mirror | Toggle | Off | Mirror left/right halves of spectrum |

---

### 9. `bubble_wrap.html` — Bubble Wrap

**Sensory quality:** Simple, satisfying, cause-and-effect. Low stimulation. Maximum clarity
between action and result — ideal for students at early cause-and-effect stage or those who
need a break from complex stimulation.

**Visual concept:** The screen is filled with a grid of glossy bubbles. Touching a bubble
pops it with a satisfying visual and sound. When all bubbles are popped, they slowly
reinflate one by one.

**Rendering — WebGL fragment shader:**

Each pixel:
1. Determine which grid cell this pixel belongs to based on column/row settings and gap.
2. Compute distance from pixel to cell centre.
3. If within bubble radius:
   a. Intact bubble: compute specular highlight (Phong-style, light from top-left),
      rim lighting (bright at edge), translucent fill, inner shadow.
      State = 1.0 (intact), animating (0–1 pop), or 0.0 (popped).
   b. Popped state: render as flat concave disc with slight specularity.
   c. Reinflating: scale radius from 0 to full over `reinflateTime`.
4. Pop animation: scale 0.95 → 1.15 → 0.0 over `popDuration` frames.
   Particle fragments: 8–12 short velocity vectors, render as small bright points
   flying outward and fading.

Bubble state array (intact/popped/reinflating + animation timer) stored in a JavaScript
`Float32Array`, uploaded to shader as a uniform texture each frame.

**Interaction:**
- Touch a bubble: pops it if intact (no effect if already popped or reinflating)
- 5 simultaneous touches each independently pop their respective bubbles

**Reinflation:**
- 1.5 seconds after last bubble popped: begin reinflation sequence
- One bubble reinflates every `reinflateDelay` seconds, random order
- Reinflation plays a soft rising chime per bubble
- When all bubbles are intact again: brief celebratory 4-note chord

**Themes:** Classic (translucent blue/white) · Rainbow (each bubble a unique hue) · Gold (amber/honey) · Night (dark iridescent — pops reveal bright colour)

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Columns | 4 – 20 | 12 | Grid columns (reset to reflow) |
| Rows | 3 – 12 | 7 | Grid rows (reset to reflow) |
| Bubble gap | 0 – 20 | 5 | Gap between bubbles (px) |
| Pop speed | 0.5 – 3.0 | 1.0 | Pop animation speed multiplier |
| Fragment count | 0 – 20 | 10 | Particles per pop (0 = no fragments) |
| Reinflate speed | 0.1 – 2.0 | 1.0 | Reinflation animation speed |
| Reinflate delay | 0.05 – 1.0 | 0.30 | Seconds between each bubble reinflating |
| Specular intensity | 0 – 2 | 1.0 | Brightness of bubble highlight |
| Pitch spread | 0 – 1 | 0.5 | How much pop pitch varies across grid |
| Volume | 0 – 1 | 0.30 | Master audio level |

**Audio:**
- Pop: white noise burst through resonant bandpass filter (`Q = 15`), followed by fast
  exponential pitch-down sweep. Each bubble has a unique resonant frequency based on
  grid position (top-left = highest pitch, bottom-right = lowest).
- Reinflate: soft sine chime (fast attack, slow decay) per bubble.
- All-popped: 4-note major chord (root, major 3rd, 5th, octave) as sine tones, 1s duration.
- All synthesised — no audio files.

---

### 10. `lightning.html` — Lightning

**Sensory quality:** Dramatic, exciting, powerful. High stimulation. Visually spectacular
on a large projected screen. Best for students who enjoy intense sensory experiences.

**Visual concept:** Dark stormy sky. Touch points become storm nodes. Lightning bolts arc
between all active touch points using recursive midpoint displacement. Single touch: bolts
arc from finger to random screen edges. Multiple touches: a full network of bolts arcs
between all nodes simultaneously.

**Rendering — WebGL (background) + Canvas 2D overlay (bolt paths):**

Two layered canvases with identical dimensions, both fullscreen:

**WebGL canvas (background layer):**
- FBM cloud texture, slowly churning over time.
- On bolt strike: full-screen flash overlay (white → blue, exponential fade over 150ms).
- Ambient node glow: soft radial gradient (theme colour) at each active touch position.
- Underlighting: the clouds are lit from below at touch positions.

**Canvas 2D overlay (bolt layer, drawn on top):**
- Bolt algorithm: midpoint displacement.
  - Given two endpoints, subdivide: take midpoint, displace perpendicular by
    `random(-1, 1) × segmentLength × roughness`, recurse `depth` levels.
  - Result: a list of points forming a jagged path.
- Render: draw the path with `ctx.strokeStyle`, `globalCompositeOperation = 'lighter'`.
  - Main trunk: lineWidth 2.5, bright white/theme colour.
  - 2–4 branches: each splits from a random trunk point, recurses 2 fewer levels,
    lineWidth 1, 60% brightness.
- Recompute bolt every 2–4 frames (random interval) for natural flicker.
- Between strikes: persistent dim channel, lineWidth 1, alpha 0.12.
- Node orbs: filled circle at each touch point, radial gradient glow.

**Touch interaction:**
- Touch down: node orb appears; bolt begins within 200ms
- Moving touch: node and bolt origin follow finger
- Multiple touches: bolts form a network — every pair of active nodes has a bolt between them
  plus each node shoots a bolt to the nearest screen edge
- Touch up: final bright strike, node fades over 0.5s

**Themes:** Storm (white/blue, grey clouds) · Plasma (purple/cyan) · Fire (orange/red) · Neon (green, black sky)

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Bolt roughness | 0.1 – 1.0 | 0.5 | Jaggedness / deviation of bolt path |
| Recursion depth | 3 – 10 | 7 | Path detail (higher = more jagged segments) |
| Flicker rate | 1 – 8 | 3 | Average frames between bolt redraws |
| Branch count | 0 – 6 | 3 | Side branches per bolt |
| Branch length | 0.1 – 0.8 | 0.4 | Branch length as fraction of main bolt |
| Flash brightness | 0 – 1 | 0.5 | Full-screen flash intensity on strike |
| Cloud speed | 0 – 0.3 | 0.08 | Background cloud churn rate |
| Cloud density | 0 – 1 | 0.6 | Darkness/thickness of cloud layer |
| Thunder delay | 0 – 1 | 1.0 | Scale of delay between flash and thunder |
| Volume | 0 – 1 | 0.30 | Master audio level |

**Audio:**
- Touch down: crackling build-up — bandpass-filtered noise, rising pitch over 0.3s.
- Bolt strike: sharp crack (noise burst, 50ms) → low rumbling thunder (low-pass filtered
  noise, 800ms). Thunder delay proportional to distance between nodes.
- Active touch: deep ambient drone — very low frequency filtered noise, gain 0.06.
- All synthesised — no audio files.

---

### 11. `shatter.html` — Shatter

**Sensory quality:** Dramatic, cathartic, surprising. High stimulation initially, then
calming as pieces reassemble. The reassembly is deeply satisfying to watch.

**Visual concept:** The screen displays a beautiful coloured pattern (Voronoi-tessellated
stained glass or a shifting gradient). Touch causes cracks to propagate from the contact
point across the entire screen. Fragments then separate, drift off screen, and finally
fly back to reassemble — locking back into place with a flash.

**Rendering — WebGL throughout:**

**Pre-computation (on load and on theme change / reset):**
1. Generate N random seed points (N from settings, default 100).
2. Compute Voronoi diagram: for each point on a low-resolution grid, find the nearest
   seed — store result as a seed-index texture.
3. For each fragment: compute polygon vertices (by collecting edge pixels), centroid,
   area, neighbour list (seeds sharing an edge), and assign a theme colour.
4. Upload fragment geometry to GPU buffers.

**Render pass (per frame):**
- State machine per fragment: `intact` → `cracking` → `falling` → `reassembling` → `intact`.
- **Intact / cracking**: render each Voronoi polygon in its theme colour. Crack edges
  drawn as dark lines with a bright refraction highlight on one side.
- **Falling**: each fragment rendered at `centroid + velocity × t + 0.5 × gravity × t²`,
  rotated by `angularVelocity × t`. Fade alpha as fragment moves off screen.
- **Reassembling**: fragment position interpolates from current off-screen position back
  to original centroid using cubic ease-out.
- Flash effect on lock-back: bright white overlay on the fragment, decays 200ms.

**Crack propagation:**
- On touch: find the nearest fragment centroid.
- BFS over neighbour graph, one ring per `crackInterval` milliseconds.
- Mark each reached edge as cracked; render as dark jagged line + refraction highlight.
- Full screen cracked in ~600ms.

**Fragment physics (post-crack):**
- Initial velocity: directed away from touch centroid, magnitude =
  `baseVelocity × exp(−distance / falloff)`.
- Angular velocity: random ±0.05 radians/frame.
- Gravity: constant downward acceleration.
- Fragments drift off screen over ~2s.

**Reassembly:**
- Triggered 2.5s after shattering completes.
- Fragments fly back to original positions, cubic ease-out over 1.2s.
- Flash on lock-back.

**Multiple touches:** each creates an independent crack origin; BFS fronts merge at shared edges.

**Themes:** Stained Glass (jewel colours, dark lead lines) · Ice (pale blue/white, bright crack lines) · Gold (black bg, gold fragments) · Neon (black bg, glowing neon fragments)

**Settings:**

| Setting | Range | Default | Effect |
|---|---|---|---|
| Fragment count | 30 – 200 | 100 | Voronoi seed count (reset required) |
| Crack speed | 10 – 100ms | 30ms | Delay between crack ring propagations |
| Fragment velocity | 0.1 – 2.0 | 1.0 | Speed multiplier for falling fragments |
| Gravity | 0 – 0.02 | 0.006 | Downward pull on fragments |
| Rotation speed | 0 – 0.1 | 0.02 | Fragment rotation while airborne |
| Reassembly speed | 0.3 – 3.0 | 1.0 | Speed multiplier for flying back |
| Reassembly delay | 0.5 – 5.0 | 2.5 | Seconds before reassembly begins |
| Crack highlight | 0 – 2 | 1.0 | Brightness of crack refraction highlight |
| Flash intensity | 0 – 1 | 0.6 | Brightness of lock-back flash |
| Volume | 0 – 1 | 0.30 | Master audio level |

**Audio:**
- Touch down: glass resonance tone (sine ~1200Hz, fast exponential decay, 300ms).
- Crack propagation: one noise burst per crack ring, pitch descending with each ring.
- Shatter: dramatic glass smash — noise through comb filter, 200ms.
- Fragments flying: subtle whoosh (bandpass filtered noise, 500ms).
- Reassembly: ascending crystalline arpeggio (sine tones, one per ring of fragments locking back).
- All synthesised — no audio files.

---

## File List

| # | Filename | Effect |
|---|---|---|
| 1 | `fluid_sensory.html` | Navier-Stokes fluid dynamics |
| 2 | `aurora.html` | Aurora Borealis curtains |
| 3 | `galaxy.html` | Spiral galaxy gravity wells |
| 4 | `ocean.html` | Deep ocean bioluminescence |
| 5 | `lava.html` | Lava domain warp |
| 6 | `gravity_sand.html` | GPU particle sand physics |
| 7 | `kaleidoscope.html` | Real-time mirror mandala |
| 8 | `sound_visualiser.html` | Microphone-driven visuals |
| 9 | `bubble_wrap.html` | Poppable bubble grid |
| 10 | `lightning.html` | Multi-touch lightning network |
| 11 | `shatter.html` | Voronoi glass shatter |

---

## Suggested Build Order

Build and fully test one file before starting the next.

1. `bubble_wrap.html` — Simplest logic. WebGL SDF bubbles, no physics complexity.
   Good warm-up for establishing the settings panel pattern used by all files.
2. `kaleidoscope.html` — WebGL render-to-texture, polar coordinate mirroring.
3. `aurora.html` — GLSL FBM noise, touch warp uniforms.
4. `lava.html` — Double domain warp FBM. Builds on aurora shader knowledge.
5. `ocean.html` — SDF jellyfish, pressure wave rings, multiple layered effects.
6. `galaxy.html` — Analytic spiral, per-pixel gravity accumulation.
7. `lightning.html` — Midpoint displacement algorithm, dual canvas (WebGL + 2D).
8. `gravity_sand.html` — GPU ping-pong particle simulation. Most complex JS setup.
9. `fluid_sensory.html` — Full Navier-Stokes pipeline. Most complex shader pipeline.
10. `sound_visualiser.html` — getUserMedia + FFT texture + 4 visual modes.
11. `shatter.html` — Voronoi pre-computation + fragment physics + reassembly. Hardest overall.

---

## Testing Checklist (run for every file before marking complete)

**Offline**
- [ ] File opens from `file:///` in Chrome with zero console errors
- [ ] DevTools → Network tab → "Offline" mode enabled: page still fully functional
- [ ] No requests to any external domain at any point

**Display**
- [ ] Fills viewport at 16:9 landscape (primary target)
- [ ] Does not break at 4:3 or portrait orientations
- [ ] Sharp on HiDPI display (devicePixelRatio > 1)

**Touch**
- [ ] 5 simultaneous touches all register and behave independently
- [ ] No ghost touches or stuck states after rapid touch sequences
- [ ] Coordinates are correctly offset (no drift from canvas position)
- [ ] Mouse fallback works for single-pointer testing

**Audio**
- [ ] No audio before first user gesture
- [ ] Audio starts correctly on first tap after page load
- [ ] Stereo panning follows horizontal touch position
- [ ] Pitch/timbre changes with vertical position
- [ ] Audio fades gracefully when touch ends (no clicks or pops)
- [ ] Multiple simultaneous touches produce harmonious chords

**Settings**
- [ ] Settings panel opens and closes with smooth animation
- [ ] Every slider applies its effect live with no perceptible lag
- [ ] Current value display updates as slider moves
- [ ] Settings persist after page refresh (localStorage)
- [ ] Reset button cleanly restores initial animation state
- [ ] Theme buttons do not accidentally trigger canvas interaction

**Performance**
- [ ] Sustained 60fps on Intel Iris Xe or equivalent integrated GPU
- [ ] No frame drops during rapid multi-touch input
- [ ] No memory leaks over 10 minutes of continuous use
  (Chrome DevTools → Memory → Heap size stays stable)
