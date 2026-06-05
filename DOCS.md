# SKYLINE STACK — Technical Documentation

> Read this file to understand the **whole game and where to change it — without reading code**.
> Single file: `index.html` (zero-build, Three.js via importmap CDN). All balance numbers live in **§14 (`CONFIG`)**.
> Approx line numbers are hints — always grep the function/marker name to locate.

---

## 0. FEATURE STATUS (read first)

| Feature | Status | Where (function / marker) |
|---|---|---|
| Core loop — slide → tap-drop → slice overhang | ✅ | `update()` (slide), `drop()` (commit/slice) |
| Alternating slide axis (X / Z each level) | ✅ | `spawnMoving()` (`S.axis`) |
| PERFECT drop — no trim + grow-back + combo | ✅ | `drop()` (`perfect` branch) |
| Pitch ladder (rises per perfect, resets on miss) | ✅ | `Audio.perfect()` + `Audio.ladder` |
| Perfect-streak heat — colour gradient → coral + screen shift | ✅ | `blockColor()`, `setHeat()`, `#heat` overlay |
| Sliced debris + slice particle shards (fall + fade, disposed) | ✅ | `spawnDebris()`, `spawnShards()`, `update()` debris loop |
| Game over (miss / sliver too small) | ✅ | `topple()` |
| Camera pans up smoothly as tower grows | ✅ | `update()` camera block (`camTargetY`) |
| Near-fall wobble on a thin sliver | ✅ | `drop()` (`WOBBLE_AT`) + `update()` wobble |
| Geometry/material disposal below view (no leaks) | ✅ | `cullBelow()` |
| Start-speed modes (Chill / Classic / Rush) | ✅ | `CONFIG.MODES`, `#modePills` |
| Subtle bloom + premium lighting + env | ✅ | `boot()` (composer, lights, PMREM) |
| CDN / WebGL fail guard | ✅ | `boot().catch` → `#fatal` overlay |
| Auto quality drop (lowFX) when FPS sags | ✅ | `fpsWatch()` |
| Synthwave music loop (tempo rises with height) | ✅ | `Music` |
| SFX + mute + haptics | ✅ | `Audio`, `haptic()`, `#muteBtn` |
| localStorage (best height + longest perfect streak) | ✅ | `Save`, keys `skyline-stack.*` |
| First-run onboarding (How to Play) | ✅ | `#how`, `onboarded` flag |
| Pause / resume | ✅ | `pauseGame()` / `resumeGame()` |
| `?shot=` screenshot hook + dev helpers | ✅ | bottom of `<script>` |

**Backlog (not built):** online leaderboard, daily reward, PWA/offline (intentionally omitted — see §2).

---

## 1. Overview & Concept

### Market context (research 2026-06-05)
- **Genre hot:** stacking is a core hyper-casual mechanic; Ketchapp's *Stack* (70M+ installs, 4.0★) is the canonical reference, praised as *minimalist, meditative, "watching pieces align perfectly is a rewarding visual payoff."* (CrazyGames, App Store, Capermint mechanics roundup).
- **Theme/style hot:** neon/synthwave reads premium and fits the QUANG ARCADE family; classic Stack uses a muted gradient — there's room to make it vivid.
- **2026 shift:** pure hyper-casual is giving way to **hybrid-casual** — the same one-tap simplicity but with light progression/reward depth (dev.to, gamegrowthadvisor).
- **Nearest competitor:** Ketchapp *Stack* — strong feel, but **no juice/progression beyond the high score** and a flat palette; *Stack Fall* is a different (ball-smash) mechanic.
- **Hook:** *"Stack, but it gets hot."* Chain **PERFECT** drops and the tower grows back, the pitch ladder climbs, and the skyline shifts from cool cyan to **white-hot coral** — the hybrid-casual reward layer the originals lack, plus start-speed modes.

### Concept
- **Genre:** 3D one-tap tower stacker (hyper-casual / hybrid-casual).
- **Core loop (1 sentence):** *"A block slides above the tower — tap to drop it, the overhang is sliced off and the rest becomes the new top; keep stacking as it speeds up; higher = better."*
- **Win / Lose:** No win — endless. **Lose** when a drop misses the tower (or the remaining sliver is too small).
- **Fantasy:** the calm, meditative flow of nailing perfect drops and watching the skyline glow hotter.
- **2D or 3D + why:** **3D (Three.js)** — depth, the angled camera panning up, and slices tumbling off as debris are the whole "wow"; a stacker is the textbook case for 3D.
- **Layout (mobile #1):** **A — Mobile-frame.** Portrait phone frame `min(100vw,480px)` centered, letterbox + gradient-mesh on desktop. A vertical tower needs no wide view → one layout for both, no responsive breakpoints.

## 2. Tech stack
- **Render:** Three.js **r0.169.0** via importmap CDN (jsDelivr). `EffectComposer` + `UnrealBloomPass` + `OutputPass`; `PMREMGenerator` + `RoomEnvironment` for image-based lighting.
- **Build:** zero-build, single `index.html`, ES modules inline, no framework/bundler/npm.
- **External resources:** Google Fonts (Orbitron + Space Grotesk) + Three.js CDN only. Everything else inline.
- **Storage:** `localStorage` (`skyline-stack.*`). **Audio:** WebAudio synth (no audio files).
- **No service worker / PWA** — the game needs no offline mode, so the SW is omitted to avoid stale-cache traps (per guide §6c).

## 3. Lifecycle / State machine
`S.mode ∈ menu → playing → (paused) → gameover → menu/playing`
- **menu:** pick a start-speed mode, view best height + longest streak.
- **playing:** the active block slides; tap drops it. rAF render loop runs always; logic only advances while `playing`.
- **paused:** slide frozen, scene still renders; music stops.
- **gameover:** the missed block tumbles, then the end panel shows (height + best + perfects + longest streak).
- Plus **fatal** overlay if the 3D engine fails to load (CDN/WebGL guard).

> **Loop model:** this is a continuous real-time action game, so the loop is **rAF + delta-time** with `dt` clamped to ≤ 0.05 s (`loop()`), not a fixed-tick grid stepper. The slide is constant-speed; the camera lerps for smoothness. (Fixed-tick is reserved for deterministic grid games like snake/tetris.)

## 4. Gameplay & rules
- **World:** a column of cuboids. The base sits at the origin; each block centre is at `y = (level+0.5)·BLOCK_H`. A block stores `{x, z, sx, sz}` (independent X/Z footprint, fixed height).
- **Slide:** the active block oscillates at constant speed between `−AMP` and `+AMP` around world centre on the current axis (the non-slide axis matches the block below).
- **Drop (tap / Space / click):** compare the block to the one below on the slide axis:
  - `delta = movingCentre − belowCentre`, `overlap = belowSize − |delta|`.
  - `overlap < MIN_SLIVER` → **tower toppled** (game over).
  - `|delta| ≤ PERFECT` → **PERFECT** (see §7.2).
  - otherwise → **trim:** new footprint on that axis `= overlap`, recentred on the overlap region; the overhang (`|delta|` wide) spawns as falling debris **plus a spray of particle shards at the cut plane**.
- **Axis alternates** every level: odd levels slide on **X**, even on **Z** (`spawnMoving()`), so the tower is carved on both axes.
- **Height = score** = number of blocks placed above the base.

---

## 5. LEVEL STRUCTURE ⭐

### 5.1 Level model
- [x] **Endless + ramp.** One infinite run; "level" = current height. No discrete stages — endless height **is** the progression. Difficulty rises continuously with height (speed) and emergent narrowing (accumulated trims).

### 5.2 Where levels are defined
- There is no `LEVELS[]` table — every level is generated by the same rules. The only per-run input is the **start-speed mode** (`CONFIG.MODES`, §6.1).
- Per-height behaviour comes from formulas:
  - **Slide speed:** `v(height) = min(mode.base + height·mode.perLevel, mode.max)` — `update()`.
  - **Block colour:** `hue = hueStart + height·hueStep`, blended toward `perfectHue` by streak heat — `blockColor()`.

### 5.3 Representative milestones (endless)
| Height | What changes | Feel |
|---|---|---|
| 0 | Base block, cool cyan (`hue 200`) | calm start |
| 1–10 | Slide near `base` speed; wide blocks, forgiving | learning |
| ~10–25 | Speed climbing toward `max`; colours walk cyan→violet | flow |
| ~25–40+ | Speed near `max`; tower often narrowed by trims | tension |
| any streak ≥2 | Heat overlay + hotter block hues + rising pitch + grow-back | reward spike |
| streak ≥ `STREAK_MAX` (8) | Max heat: white-hot coral skyline, pitch ladder topped out | peak juice |

### 5.4 Progression & milestones
- Every successful drop: `height++`, colour advances, speed recomputed.
- The emotional "milestone" is **the perfect streak**, not a biome change: each consecutive PERFECT pushes colour, pitch, screen heat and grow-back further (§7.2).

---

## 6. DIFFICULTY STRUCTURE ⭐

### 6.1 Start-speed modes (`CONFIG.MODES`)
Difficulty = how fast the block slides (and how fast that accelerates). Footprint, perfect window and slice rules are identical across modes.

| Mode | base speed (u/s) | perLevel (u/s per height) | max speed (u/s) |
|---|---|---|---|
| **Chill** | 2.6 | 0.045 | 7.0 |
| **Classic** (default) | 3.4 | 0.065 | 9.0 |
| **Rush** | 4.8 | 0.090 | 12.5 |

### 6.2 In-run ramp
- **Speed:** `v = min(base + height·perLevel, max)`. Chill reaches its cap ~height 98, Classic ~height 86, Rush ~height 85 — but most runs end well before the cap, so the felt ramp is the steady early climb.
- **Emergent difficulty:** off-centre drops shrink the footprint; a narrower top is harder to hit, so mistakes compound — the classic Stack tension. PERFECT drops fight back by growing the footprint (`GROWBACK`).
- **Curve intent:** anyone can place the first ~10 (wide, slow). Skill shows in holding perfects to keep the tower wide while speed rises.

### 6.3 To change difficulty → see §15.2.

---

## 7. SCORING ⭐

### 7.1 Score sources
| Action | Points | Formula |
|---|---|---|
| Place any block (perfect or trimmed) | +1 height | `S.height = level` |
| Perfect drop | (no extra points) | counted in `S.perfects`; drives juice, not score |

**Score = tower height = number of blocks above the base.** Deliberately pure (matches the genre); the reward for skill is *staying alive longer & wider*, plus the perfect-streak spectacle — not a number multiplier.

### 7.2 PERFECT & streak (the "combo")
- **Trigger:** `|delta| ≤ PERFECT` (0.18 world units) at the moment of the drop.
- **Effects, all in `drop()`:**
  - No trim; footprint **grows back** by `GROWBACK` (0.18), capped at `BASE_W`.
  - `S.streak++`, `S.perfects++`.
  - **Pitch ladder:** `Audio.perfect(streak)` plays the next note up `Audio.ladder` (C5→E6), capped at the top note.
  - **Heat** (`heat = min(streak/STREAK_MAX, 1)`): block hue blends toward coral, saturation rises; `#heat` screen overlay fades in; placed block gets a faint emissive glow.
  - White flash + `PERFECT ×N` popup + double haptic + small camera shake + a celebratory **shard burst** (`spawnShards`, hot colour).
- **Streak breaks** (set to 0) on any trimmed (non-perfect) drop.
- **Longest streak** is saved (`skyline-stack.bestStreak`).

### 7.3 Best / record
- `best` (max height) and `bestStreak` (longest perfect streak) saved to `localStorage`; shown on the menu and game-over panel.

### 7.4 To change scoring → see §15.3.

## 8. Economy
None. No coins/shop — kept pure hyper-casual.

## 9. Power-ups / evolution
None as discrete items. The **perfect-streak heat** is the only "power" system (grow-back + visual/audio escalation); see §7.2.

## 10. Audio
- **SFX map** (`Audio`): `drop()` soft "tok" on a trim, `slice()` chip on overhang cut, `perfect(streak)` rising pitch-ladder chime, `over()` descending game-over tumble.
- **Music** (`Music`): gentle synthwave bed — lookahead scheduler, arpeggio + bass/pad; **tempo rises with height** (`bpm = 84 + min(height·0.9, 46)`). Starts on play, stops on pause/menu/gameover.
- **Mute:** `#muteBtn` (🔊/🔇), persisted. Per-channel toggles (Music / Sound FX / Haptics) in the pause panel.
- iOS/Android: `Audio.ensure()` is called inside the first gesture so the context resumes (no silent audio).

## 11. Controls
- **Desktop:** click anywhere / **Space** to drop · **Esc** or **P** to pause/resume.
- **Mobile:** **tap anywhere** to drop. Overlays use `pointer-events:none` when hidden to prevent tap-leak; the canvas uses `touch-action:none`.
- Haptics fire only after the first real gesture (`gestured` flag) — avoids the blocked-`vibrate` console warning.

## 12. State object `S` (main fields)
```
mode        'menu' | 'playing' | 'paused' | 'gameover'
modeKey     'chill' | 'classic' | 'rush'   (selected start-speed)
spd         active mode object {base, perLevel, max}
height      current tower height (= score)
perfects    perfect drops this run
streak      current consecutive-perfect count (drives heat/pitch)
best, bestStreak   persisted records
tower[]     placed blocks {mesh, x, z, sx, sz} (base at [0])
debris[]    falling sliced pieces {mesh, vx, vy, vz, rx, rz, life}
move        active sliding block {mesh, x, z, sx, sz, lvl} | null
axis        'x' | 'z'  (current slide axis); dir ±1 (slide direction)
shake, wobble, wobbleSign, engineReady
```

## 13. localStorage keys
| key | meaning |
|---|---|
| `skyline-stack.best` | best height (record) |
| `skyline-stack.bestStreak` | longest perfect streak |
| `skyline-stack.muted` | master mute |
| `skyline-stack.music` / `.sfx` / `.haptic` | channel toggles |
| `skyline-stack.onboarded` | first-run How-to-Play shown |
| `skyline-stack.plays` | games played |

---

## 14. BALANCE NUMBERS (single source of truth) ⭐
All in the `CONFIG` object near the top of the `<script type="module">` in `index.html`.
```javascript
const CONFIG = {
  BLOCK_H:   0.62,    // block thickness (world units, Y)
  BASE_W:    3.2,     // starting square footprint
  AMP:       3.05,    // slide half-range around centre (~BASE_W → a worst-time tap can still miss)
  PERFECT:   0.18,    // |delta| ≤ this = PERFECT
  GROWBACK:  0.18,    // footprint restored toward BASE_W per perfect
  MIN_SLIVER:0.30,    // overlap below this = game over
  WOBBLE_AT: 0.95,    // overlap below this (and not perfect) = near-fall wobble
  STREAK_MAX:8,       // streak at which heat / pitch ladder maxes out
  MODES: {
    chill:   { base:2.6, perLevel:0.045, max:7.0,  label:'Chill'   },
    classic: { base:3.4, perLevel:0.065, max:9.0,  label:'Classic' },
    rush:    { base:4.8, perLevel:0.090, max:12.5, label:'Rush'    },
  },
  COLOR: { hueStart:200, hueStep:4.2, perfectHue:14 },         // cyan→ walk, coral on heat
  CAM:   { dist:21, fov:30, dirX:1, dirY:0.86, dirZ:1, lookAhead:-1.9, follow:6.5 },
  VISIBLE_DEPTH: 16,  // blocks kept below the top before culling+disposing
  GRAVITY: 26,        // debris fall accel
};
```
Also tunable: `Audio.ladder` (pitch-ladder notes), `#heat` opacity scale in `setHeat()`, bloom args in `boot()` (`0.45, 0.4, 0.84`).

---

## 15. HOW-TO recipes ⭐

### 15.1 Add a new start-speed mode
1. Add an entry to `CONFIG.MODES` (e.g. `insane:{base:6.2, perLevel:0.11, max:15, label:'Insane'}`).
2. Add a `<button class="pill" data-mode="insane">Insane<small>…</small></button>` to `#modePills` in the HTML.
3. Update **§6.1** table here.
4. Test each mode (`?shot=play` after selecting), commit with DOCS.

### 15.2 Tune difficulty
1. Edit `CONFIG.MODES` (speed `base` / `perLevel` / `max`) and/or `AMP`, `PERFECT`, `GROWBACK`, `MIN_SLIVER`.
2. Update **§6** tables to match.
3. Play each mode; check the curve feels right (easy entry, rising tension).
4. Commit code + DOCS together.

### 15.3 Change scoring
1. Score is `S.height`. To add a points system, change where `S.height`/score is set in `drop()` and the HUD/`gameover` display (`updateHUD`, `topple`).
2. Update **§7** here.
3. Verify the HUD + game-over numbers + `best` persistence.
4. Commit code + DOCS together.

### 15.4 Tune the perfect-streak feel (colour / pitch / heat)
1. `COLOR` (hue walk + coral target), `STREAK_MAX` (how fast heat builds), `Audio.ladder` (notes), `GROWBACK`.
2. `setHeat()` opacity scale + `#heat` CSS gradient control the screen shift.
3. Update **§7.2** + **§5.3** if behaviour changes.
4. Commit code + DOCS together.

---

## 16. Update history
- **2026-06-05** (initial): SKYLINE STACK v1 — 3D Stack core, perfect-streak heat (colour + pitch ladder + grow-back), 3 start-speed modes, sliced debris + slice particle shards, camera pan, bloom, synthwave, full shell + QA evidence.

> **Last updated:** 2026-06-05 · branch `main` · single-file `index.html`.
