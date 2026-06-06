# Core Workout Timer

A progressive web app (PWA) for guided core workouts — warm-up, core training, and cool-down — with a smart timer, per-exercise set/side/round tracking, audio cues, and YouTube video references for every exercise.

**Live:** Deploy to [Cloudflare Pages](https://pages.cloudflare.com) by uploading `index.html` — no build step needed.

---

## Features

- **Three-phase workout flow** — Warm-up → Core → Cool-down with phase nav, skip buttons, and smooth transitions
- **PT-correct set/round/side logic** — each exercise knows whether it uses sets, rounds, or bilateral sides (e.g. Side Plank counts 2 sides, Bridge counts 3 sets, Cat-Cow counts 1 round)
- **Smart audio** — soft tick every second, medium beep for last 10 sec, loud + vibration for last 3 sec; speech announces the next exercise name and whether to switch sides or start a new set
- **YouTube thumbnails + links** — every exercise shows a reference image (YouTube thumbnail) that links to a tutorial video
- **Live elapsed timer** — tracks total session time across all phases
- **Workout completion screen** — shows time taken and exercises completed per phase
- **PWA installable** — works offline after first load, installable to home screen on iOS/Android (requires HTTPS)
- **Screen wake lock** — keeps screen on during workout (HTTPS only)
- **Haptic-only mode** — silent vibration mode for gym use
- **Landscape layout** — media and timer sit side by side on phones in landscape
- **Safe-area aware** — notch/home-bar padding on iPhone

---

## Exercises

### Warm-up (4)
Cat-Cow · Dead Bug · Pelvic Tilt · Lying Hip March

### Core Workout (11)
| Beginner | Intermediate | Advanced |
|---|---|---|
| Bridge (3 sets) | Plank (3 sets) | Mountain Climber |
| Crunch | Warrior Crunch | Side Plank with Rotation (2 sides) |
| Supine Toe Tap | Bird Dog Elbow to Knee (2 sides) | Turkish Get-Up (3 sets) |
| Bird Dog | | |
| Bicycle Crunch (3 sets) | | |

### Cool-down (5)
Child's Pose · Supine Twist (2 sides) · Knee-to-Chest (2 sides) · Hip Flexor Stretch (2 sides) · Seated Forward Fold

---

## Deployment

### Cloudflare Pages (recommended, free)
1. Go to [pages.cloudflare.com](https://pages.cloudflare.com)
2. Create a project → **Direct Upload**
3. Drag and drop `index.html`
4. Click **Deploy site**

Your app will be live at `your-project.pages.dev` in ~30 seconds with HTTPS, which unlocks the screen wake lock and PWA install.

### Local development
Just open `index.html` in a browser. Timer, audio, and all UI features work on `file://`. Wake lock and PWA install require HTTPS.

---

## Architecture

Single self-contained HTML file — no framework, no build tool, no dependencies.

```
index.html
├── <style>          CSS (light theme, orange accent, responsive)
├── Exercise data    warmupExercises / workoutExercises / cooldownExercises arrays
├── State machine    phase → warmup | workout | cooldown | done
├── Timer engine     requestAnimationFrame tick loop
├── Audio engine     Web Audio API beep() + SpeechSynthesis speak()
├── PWA setup        Inline manifest + blob Service Worker
└── Event listeners  All UI interactions
```

### State object
```js
{
  phase,        // warmup | workout | cooldown | done
  index,        // current exercise index within phase
  reps,         // { "phase:name": setsLeft } — per-exercise counter
  completed,    // { "phase:name": true } — completed exercises
  remaining,    // seconds left on current timer
  mode,         // exercise | rest
  restKind,     // rep | exercise
  restUnit,     // rounds | sets | sides — drives speech on rest end
  totalElapsed, // cumulative session seconds
  sessionStart  // Date.now() when timer last started
}
```

### Exercise schema
```js
{
  phase: "warmup" | "workout" | "cooldown",
  level: "beginner" | "intermediate" | "advanced",  // workout only
  sets:  Number,   // default counter value (1, 2, or 3)
  unit:  "rounds" | "sets" | "sides",
  name, target, seconds,
  image, video,  // YouTube thumbnail URL + watch URL
  cues:  [String, String, String]
}
```

---

## Credits

Exercise content based on [Healthline — Best Core Exercises](https://www.healthline.com/health/best-core-exercises).  
Video tutorials sourced from YouTube.

---

## Licence

MIT — see [LICENSE](LICENSE)
