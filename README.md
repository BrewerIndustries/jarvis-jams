# Jarvis Jams

Browser-based music sandbox — a step-sequencer beat machine and a guitar tuner / tone generator. No backend, no login, no install. Everything runs in the Web Audio API and persists to localStorage.

Part of the Jarvis constellation.

## Tools

### Beat Machine
- 8 instrument rows: Kick, Snare, Hi-Hat, Open Hat, Clap, Tom, Rim, Cowbell
- Click-to-arm step grid — lookahead scheduler (not setInterval) for tight timing
- BPM 40–300 + tap-tempo (averages last 8 taps)
- Beats per measure 2–8 · Step resolution 1/4, 1/8, 1/16, 1/32 (hot-swap while playing)
- Swing 0–100%
- Named patterns saved to localStorage · Export loop as WAV (OfflineAudioContext)

### Guitar Tuner
- Click a string to play its reference tone (sine or guitar-like harmonics)
- 8 preset tunings: Standard, Drop D, Open G, Open D, DADGAD, Drop C, Half-step down, Full-step down
- Custom tuning editor — note + octave picker per string, live Hz preview
- Save named tunings to localStorage · Export as JSON

## Environments

| Env  | URL                                   |
| ---- | ------------------------------------- |
| Prod | https://jarvis-jams.dabrewer.dev/     |
| Dev  | https://jarvis-jams.dabrewer.dev/dev/ |

Hosted on GitHub Pages. The workflow lives on `dev`; it publishes `main` → `/` and `dev` → `/dev/` on every push to dev.

## Workflow

Work on `dev`. Push freely. Promote to `main` only via an approved PR.

```bash
git push origin dev   # triggers Pages deploy automatically
```

## Local development

```bash
python3 -m http.server 3000   # or use the launch.json entry in .claude/launch.json
# open http://localhost:3000
```
