# CLAUDE.md — project rules for Touch

Durable conventions for this repo. (Task state / where to resume lives in `NEXT_SESSION.md`.)

## What this is
A suite of standalone, offline-capable, fullscreen **multi-touch sensory-room animations** for a
touch projector. Each animation is a **single self-contained `.html`** — no build step, no
dependencies, no network requests at runtime. Files must work both from GitHub Pages and offline
from `file://` (USB stick).

- Repo: github.com/Magnatronic/touch · branch `main` · live: magnatronic.github.io/touch/

## Hard rules (don't break these)
- **No external dependencies.** No CDNs, no `<script src>`, no `import` from URLs, no network calls.
  All CSS/JS/GLSL inline in the one `.html`.
- **Never hand-edit a `FRAMEWORK-*` region** in an animation file. Edit the framework only in
  `template.html`, then run `node sync-framework.js` to propagate it. Verify the script reports the
  files as updated.
- Each animation only customises its **ANIMATION block** (`const Anim = {...}`). Keep the `Anim`
  contract: `themes, defaults, schema, sectionLabel, init(canvas), resize(w,h),
  splat(x,y,dx,dy,color,opts), frame(dt), reset()`. `splat` coords are 0..1 with **y up**.

## Architecture
Framework (shared, identical everywhere) provides: settings + localStorage persistence, colour
system (themes, swatches, lock, background), audio engine (9 voices, 4 scales, reverb/echo/chorus,
master limiter), 5-touch input with pointer pruning, the ≡ menu → Appearance/Sound panel, DPR
sizing, WebGL context-loss recovery, and the render loop. The animation supplies visuals + themes
+ its own settings via the `Anim` object.

## Workflow
- **Plan non-trivial work before coding**; get approval, then build.
- After framework edits: `node sync-framework.js`.
- New animation: copy `template.html` → `name.html`, replace ANIMATION block, then in `index.html`
  set that tile's `file:` from `null` to the filename.
- **Verify before pushing:** syntax-check the `<script>` (`node --check`), open the preview, and
  test 5 simultaneous touches + Reset settings (localStorage can mask new defaults).
- Commit working states often (they're the rollback points). End commit messages with the
  `Co-Authored-By` trailer used in existing commits. Push after each logical change is fine.
- CRLF/LF warnings on Windows commits are harmless.

## Constraints to honour
- 60fps target on mid-range integrated GPU (Intel Iris Xe) at 1080p.
- Up to 5 independent touches; each its own colour + audio voice.
- All audio synthesised via Web Audio API (no files); audio only starts on a user gesture.
- No secrets/keys in any repo file (repo is public).
