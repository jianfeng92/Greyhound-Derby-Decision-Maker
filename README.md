# Greyhound Derby 🏁

A single-file web app for picking a winner from a list of names — up to 12 names race as greyhounds, first past the post wins. Built for group decision-making where you want the outcome to feel fair *and* look exciting.

**Live demo:** _(add your hosted URL here once deployed)_

---

## Stack

No frameworks, no bundler, no build step, no package.json. It's one HTML file containing HTML, CSS, and vanilla JavaScript.

| Piece | Choice | Why |
|---|---|---|
| Structure/logic | Vanilla JS (ES5-leaning syntax) | No framework overhead for 12 animated DOM elements at 60fps |
| Fonts | Google Fonts CDN (`Oswald`, `Inter`, `Space Mono`) | Loaded via `<link>` — requires internet access when hosted |
| Randomness | `mulberry32` seeded PRNG | Deterministic + reproducible; the seed is shown to the user |
| Audio | Web Audio API, fully synthesized | No licensed/sampled audio, so no copyright or asset-loading concerns |
| Animation | `requestAnimationFrame` + CSS transforms | Real delta-time physics, framerate-independent |
| Persistence | None — entirely client-side, stateless between reloads | No backend, no database, nothing to configure |

## How it's built

**Screens.** One page, two `<div>`s toggled via `display: none/block` — no router. `setup-screen` collects names/duration/seed; `race-screen` renders the track and runs the animation.

**Fairness model.** This was the core design constraint: the picker needed to be *actually* fair, not just look random.
- A seed is generated with `crypto.getRandomValues` and shown on screen (`GD-XXXXXXXX`). The same seed always reproduces the same race — "Race again" rolls a new seed on purpose; you could add a "replay this seed" feature if you want strict determinism.
- Lane order is shuffled using the seeded RNG, so trap position carries no information.
- Every dog draws from the **same** distribution for its persistent pace multiplier (`paceMult`) and its surge/fade event chances. Fairness comes from symmetry — every runner is statistically identical — not from artificially equalized speed. That's also *why* the race isn't a photo finish every time: real variance in outcome is what makes the RNG trustworthy rather than suspicious.

**Physics loop (`loop()`).** Runs on `requestAnimationFrame`, computes real delta-time (clamped to 50ms so a backgrounded tab doesn't cause a jump), and updates each dog's position based on `baseSpeed × paceMult × boostMult × (wobble + noise)`. `boostMult` implements sustained 0.5–1.4s surge/fade windows so speed changes are visually readable, not just noise.

**Duration handling.** The slider (10–30s) sets an *expected* finish time, not a guaranteed one — forcing an exact clock would mean secretly adjusting speed near the end, which breaks the fairness guarantee. This tradeoff is intentional, not an oversight.

**Audio engine.** All sound is generated with `OscillatorNode`/`AudioBufferSourceNode`, not sampled files:
- `ensureAudio()` creates/resumes the `AudioContext` inside the "Start Race" click handler specifically, because browsers block audio until a user gesture — this is the one gesture guaranteed to happen.
- `playTone`, `playKick`, `playNoiseBurst` are the low-level synth primitives.
- Countdown beeps, start horn, a looping arpeggio+kick BGM (`startBgm`/`stopBgm`), lead-change pings, a crowd-cheer swell, and a winner fanfare are all composed from those primitives.
- A 🔊/🔇 toggle (fixed top-right) mutes via the master `GainNode` — it doesn't tear down the audio graph, just silences it.

**Color/contrast handling.** Trap colors follow real greyhound-racing jacket conventions (red, blue, white, black, etc.), extended to 12. Because some of those colors are very dark, raw trap color is never used as *text* color — only as a small ring-protected dot swatch or a contrast-corrected accent (`visibleAccent()`), so trap 4 (near-black) stays legible against the dark UI.

## Browser support

Needs a modern browser with Web Audio API support (all current Chrome/Firefox/Safari/Edge). If Web Audio is unavailable, `ensureAudio()` no-ops and the app runs fully silent rather than breaking.

## Running locally

No install needed:

```bash
open greyhound-derby.html    # macOS
# or just double-click the file
```

If you want it served over `http://` instead of `file://` (some browsers restrict audio/fonts on `file://`):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/greyhound-derby.html
```

## Deploying

Static file, so any static host works. Two easy options:
- **GitHub Pages** — rename the file to `index.html`, push, enable Pages in repo settings.
- **Netlify Drop** — drag the (renamed) file onto app.netlify.com/drop for an instant URL.

## User instructions

**Setup**
1. Add 2–12 names (each becomes a numbered trap/greyhound).
2. Set the target race length with the duration slider (10–30s).
3. A fairness seed is generated automatically — reroll it if you want, or leave it; the same seed always produces the same race.
4. Click **Start Race** (this also unmutes audio on first play, per browser rules).

**During the race**
- Countdown (3-2-1-GO), then the field runs.
- Live commentary and a standings panel update as positions change.
- Toggle sound with the 🔊 button, top-right, anytime.
- **← Edit names** stops the race and returns to setup.

**Finish**
- First dog to the finish line wins — no photo-finish ambiguity, ties aren't possible since position is continuous.
- The winner card shows the name, trap, and margin of victory.
- **Race again** reuses the same names with a fresh seed. **New names** returns to setup.

## License

No license file is included yet — add one (e.g. MIT) if you want to make the reuse terms explicit before sharing the repo publicly.
