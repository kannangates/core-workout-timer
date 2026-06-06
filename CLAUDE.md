# CLAUDE.md — Core Workout Timer

## Project overview
Single-file PWA workout timer (`index.html`). No framework, no build tool, no dependencies. Everything — HTML, CSS, JS, PWA manifest, service worker — lives in one file.

## Key files
- `index.html` — the entire application (≈1,590 lines)

## Architecture

### Data layer (top of `<script>`)
Three arrays define all 20 exercises:
- `warmupExercises` — 4 exercises, all `sets:1`
- `workoutExercises` — 11 exercises, filtered by `state.level` at runtime
- `cooldownExercises` — 5 exercises

Each exercise object has: `phase, sets, unit, name, target, seconds, image, video, cues[]`

**Critical field — `unit`:** drives counter label AND speech announcements
- `"sets"` — multi-set exercises (Bridge, Plank, Bicycle Crunch, Turkish Get-Up)
- `"sides"` — bilateral exercises where user does each side separately (Bird Dog Elbow to Knee, Side Plank with Rotation, Supine Twist, Knee-to-Chest, Hip Flexor Stretch)
- `"rounds"` — single timed flows (Cat-Cow, Child's Pose, Seated Forward Fold)

### State machine
```
warmup → workout → cooldown → done
```
State is persisted to `localStorage` key `core-workout-timer-state-v2` on every tick and interaction.

**Important state fields:**
- `restUnit` — stored in `startBreak()`, read by `handleRestFinished()` to announce "Switch to the other side." / "Next set." / "Next round."
- `sessionStart` — `Date.now()` snapshot; used to compute live elapsed time without adding to `totalElapsed` until session ends

### Timer loop
`requestAnimationFrame(tick)` — subtracts elapsed ms from `state.remaining`, calls `render()` and `save()` each frame. Stops when `state.running = false`.

### Key functions to know
| Function | What it does |
|---|---|
| `handleExerciseFinished()` | Decrements sets counter; if > 0 calls `startBreak("rep")`; else advances exercise or phase |
| `handleRestFinished()` | Reads `state.restUnit` → correct speech, then calls `startCurrent()` |
| `startBreak(kind, nextName, unit)` | Sets rest mode, stores `state.restUnit`, speaks contextual message |
| `goToPhase(phase, msg)` | Resets index/remaining/mode, speaks transition message |
| `showCompletion()` | Has double-call guard (`if phase === "done" return`); shows overlay with stats |
| `resetAll()` | Full state reset including totalElapsed, sessionStart, reps, completed |
| `withTransition(fn)` | Pauses ticker, fades hero, runs `fn()`, fades back in — prevents race conditions |
| `render()` | Idempotent full UI update; always removes `hidden` from skip button before re-evaluating |

### Audio system
- `beep(freq, dur, vol)` — Web Audio API, respects `soundMode !== "on"`
- `vibe(pattern)` — navigator.vibrate, respects `soundMode === "off"` (fires in "haptic" mode too)
- `speak(msg)` — SpeechSynthesis, respects `soundMode !== "on"`
- Sound cycle: on → haptic → off

### PWA
- Manifest injected as `data:application/manifest+json` URI
- Service worker registered via `URL.createObjectURL(blob)` — works on HTTPS/localhost, silently skips on `file://`

## Common tasks

### Adding an exercise
Add an object to the appropriate array (`warmupExercises`, `workoutExercises`, `cooldownExercises`). Required fields: `phase, sets, unit, name, target, seconds, image, video, cues`.

For `unit: "sides"`, set `sets: 2`. The break announcement will automatically say "Switch to the other side."

### Changing rest duration
Edit `const REST_SECS = 7;` near the top of the script.

### Changing workout level default
Edit `level: "beginner"` in the initial `state` object.

### Updating theme colours
All colours are CSS custom properties in `:root`. The accent is `--accent: #ff6d00`.

## Edge cases already handled
- First visit (no localStorage) — init defaults cover all fields
- Reload mid-rest — `state.mode` and `state.remaining` restored from localStorage
- App reopen after completion — `phase === "done"` triggers full reset to warmup on init
- Double-tap Done/Next at end — `showCompletion()` guard prevents double execution
- Image load failure — error handler shows "Image could not load" fallback
- `speechSynthesis` unavailable — `speak()` checks before calling
- `AudioContext` autoplay block — resumed on first user interaction
- Skip button hidden after "Start Again" — `render()` always calls `classList.remove("hidden")` before evaluating phase
