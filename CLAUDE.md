# Jarvis Jams

Browser-based music sandbox. Three tabs: a **step-sequencer beat machine**, a
**guitar tuner / tone generator**, and a **modular synth Rack** (drag-and-drop
oscillators/filters/effects plus special modules — Joog, Abacus, Swells, Capture,
DFAM, Sampler, Looper, etc.). All audio runs client-side on the Web Audio API; no
backend, no login, no build. State persists to localStorage. Part of the Jarvis
constellation.

## Tech
- **Single file: `index.html`** (~6400 lines). One `<style>` block, one `<script>`
  block. **Vanilla JS — no React, no framework, no CDN, no build step.**
- Web Audio API (`AudioContext` live playback, `OfflineAudioContext` for WAV export).
- Optional Web MIDI API for the keyboard; `getUserMedia` for the Capture module.
- Persistence: `localStorage`.

## Code map (all inside the one `<script>` in index.html)
- **Tabs / panels** — `.tab-btn[data-tab]` → `#panel-beat`, `#panel-tuner`,
  `#panel-rack`. `panel-keys` and `panel-joog` are created dynamically. Active tab
  saved as `jj-active-tab`.
- **Beat machine** — grid state in `grid[row][step]`, 8 instrument rows. Transport
  `play()`/`stop()`. Audio timing uses a **lookahead scheduler**, not setInterval on
  audio: `scheduler()` runs `setTimeout` every `SCHED_INTERVAL` (25ms) and schedules
  every step whose `nextNoteTime < ctx.currentTime + LOOKAHEAD` (0.1s); a separate
  `requestAnimationFrame` (`updatePlayhead`) drives the visual playhead. Constants at
  the `let isPlaying…` / `const LOOKAHEAD=0.1, SCHED_INTERVAL=25` lines (~1378).
- **Tuner / tone generator** — `playTone()`/`stopTone()`, `activeTones[]`, string
  reference tones, preset + custom tunings.
- **WAV export** — `exportWAV()`: renders the loop through an `OfflineAudioContext`,
  `encodeWAV()` writes the PCM WAV, downloads via a temporary object URL.
- **Rack** — module catalogue `MOD_DEFS` (~1609) defines every module type + its
  `defaults`; audio-node factory builds each module's graph; UI builders render the
  cards; `rackSerialize` handles save/load. The Joog synth voice is a large dedicated
  block (patch bay, 13-key pad, S+H, ladder filter) around lines 2990–3560.
- **Keyboard** — polyphonic voice pool, QWERTY mapping, Web MIDI, and a Looper.

## localStorage keys
`jj-patterns` (beat patterns) · `jj-tunings` (saved tunings) · `jj-racks` (serialized
rack instances) · `jj-knob-bindings` · `jj-active-tab`.

## Run locally
```bash
python3 -m http.server 3000   # or the .claude/launch.json entry (port 3000)
# open http://localhost:3000
```
Just opening `index.html` as a `file://` URL mostly works too, but a static server
is safer (MIDI/mic permissions, consistent behavior).

## Deploy (GitHub Pages)
Workflow `.github/workflows/pages.yml` lives on `dev` and publishes **both** versions
to one Pages site: `main` → `/` (prod), `dev` → `/dev/` (dev), custom domain
`jarvis-jams.dabrewer.dev`.
- Prod: https://jarvis-jams.dabrewer.dev/
- Dev:  https://jarvis-jams.dabrewer.dev/dev/

The workflow triggers **only on push to `dev`** (or manual `workflow_dispatch`).
Pushing to `main` does **not** auto-redeploy — re-run the workflow or push to dev.

## Git workflow
- Work on `dev`. Push to `dev` freely (it triggers the deploy).
- Promote to `main` **only via a PR the user approves**. Never fast-forward or
  reset-push `main`.
- Keep `README.md` current after any meaningful change.

## Backlog
`BACKLOG.md` holds deferred items (currently empty).

## Gotchas
- **Autoplay policy**: browsers start the `AudioContext` suspended. `getCtx()`
  resumes it (`if (audioCtx.state === 'suspended') audioCtx.resume()`), so always
  route new audio through it — audio only actually starts after a user gesture.
- **Audio timing**: schedule with `ctx.currentTime` offsets (the lookahead pattern),
  never with `setInterval`/`setTimeout` firing the sound directly, or timing drifts.
- **WAV export** re-renders the loop offline; it does not capture the live graph, so
  changes to live-only nodes won't show up in the exported file.
- Single giant file — no modules. Search within `index.html` (the `// ─── … ───`
  banner comments mark sections) rather than expecting separate files.
