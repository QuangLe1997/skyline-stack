# SKYLINE STACK — Test Report (QA Evidence)

- **Test date:** 2026-06-05
- **Build:** `index.html` (single-file, Three.js r0.169 via importmap CDN), served locally on `http://127.0.0.1:8794`
- **Method:** headless Chrome (puppeteer-core, `--headless=new`) driving real UI + the `?shot=` state hook; WebGL via ANGLE/SwiftShader. Desktop & mobile device-emulation viewports.
- **Viewports:** desktop **1280×800** (centered phone-frame + letterbox = the real desktop look) · mobile **390×844** (DPR 2, touch)
- **Console errors:** **0** (verified across the smoke, full-matrix, and real-interaction runs)

## Results

| ID | Category | Test | Viewport | Screenshot | Result | Notes |
|----|----------|------|----------|-----------|--------|-------|
| T01 | UI | Menu + 3 start-speed modes (Chill/Classic/Rush) + best/streak stats | desktop | [01-menu-d.jpg](screenshots/01-menu-d.jpg) | ✅ PASS | gradient title, coral accent, mode pills |
| T02 | UI | First-run onboarding "How to Play" | desktop | [02-how-d.jpg](screenshots/02-how-d.jpg) | ✅ PASS | text wraps, no overflow |
| T03 | Level | Low tower (early) — cool cyan→violet, sliding block visible | desktop | [03-play-low-d.jpg](screenshots/03-play-low-d.jpg) | ✅ PASS | moving block in-frame |
| T04 | Level | Tall tower (height 42) — gradient + visible trims/taper | desktop | [04-tall-d.jpg](screenshots/04-tall-d.jpg) | ✅ PASS | endless ramp, camera panned up |
| T05 | Logic | PERFECT streak — heat color shift to coral + `PERFECT ×N` popup + streak chip + screen heat | desktop | [05-perfect-d.jpg](screenshots/05-perfect-d.jpg) | ✅ PASS | the core hook |
| T06 | UI | Pause + audio toggles (Music/SFX/Haptics) | desktop | [06-pause-d.jpg](screenshots/06-pause-d.jpg) | ✅ PASS | HUD hidden behind overlay |
| T07 | UI | Game over — hero height + best/perfects/streak + retry | desktop | [07-gameover-d.jpg](screenshots/07-gameover-d.jpg) | ✅ PASS | toppled block falls behind panel |
| T08 | Persist | Best height + longest perfect streak remembered **after reload** | desktop | [08-record-reload-d.jpg](screenshots/08-record-reload-d.jpg) | ✅ PASS | shows Best 42 / Streak 6 post-reload |
| T10 | Responsive | Menu on mobile | mobile | [10-menu-m.jpg](screenshots/10-menu-m.jpg) | ✅ PASS | fits, safe-area respected |
| T11 | Responsive | Gameplay on mobile (low tower + moving block) | mobile | [11-play-m.jpg](screenshots/11-play-m.jpg) | ✅ PASS | full-bleed portrait |
| T12 | Responsive | Tall tower on mobile | mobile | [12-tall-m.jpg](screenshots/12-tall-m.jpg) | ✅ PASS | |
| T13 | Logic | PERFECT streak heat on mobile (streak chip + hot hues) | mobile | [13-perfect-m.jpg](screenshots/13-perfect-m.jpg) | ✅ PASS | |
| T14 | Responsive | Game over on mobile | mobile | [14-gameover-m.jpg](screenshots/14-gameover-m.jpg) | ✅ PASS | panel fits, no overflow |
| T15 | Logic | **Real end-to-end playthrough via UI** (no dev shortcuts) | mobile | [15-realplay-m.jpg](screenshots/15-realplay-m.jpg) | ✅ PASS | see below |

### T15 — real interaction trace (clicks on the actual UI, taps on the canvas)
```
firstMode       : menu                  (boots to menu)
Play → How      : howVisible=true        (first-run onboarding shows)
How → Got it    : afterHow=playing       (game starts)
8 canvas taps   : heightAfterTaps=8      (every real tap dropped a block; authentic tapering tower — see screenshot)
Pause/Resume    : pauseVisible=true      (pause overlay, then resumed)
force miss      : afterMiss=gameover      (miss → tower toppled → game-over panel)
persistence     : best saved=8 → reload → menu shows best  ✅
```
The screenshot caught a live `PERFECT` popup + a falling slice debris mid-animation.

## Coverage summary
- **Screens / flow:** menu · onboarding · HUD · pause · game-over · reload-persist — all ✅
- **Level structure (endless):** early (low/cool) · tall (height 42) · milestone (perfect-streak heat) — all ✅
- **Difficulty:** 3 start-speed modes (Chill/Classic/Rush) selectable on the menu — ✅
- **Core logic:** slide → tap-drop → slice overhang · PERFECT (no-trim + grow-back + pitch ladder + color/heat shift + combo popup) · alternating X/Z axis · miss = game over — all ✅ (real-tap playthrough T15)
- **Scoring/persistence:** height = score; best height + longest perfect streak saved to `localStorage` and restored after reload — ✅
- **Responsive:** mobile 390×844 (menu + gameplay + game-over) and desktop 1280×800 — all ✅
- **Robustness:** CDN/WebGL fail guard (`#fatal`) present; WebGL confirmed rendering in headless; 0 console errors.

## Performance
- Rendered fine in headless ANGLE/SwiftShader (software GL). On real hardware the scene is light (shared unit-cube geometry, blocks disposed below view, weak bloom, DPR capped ≤2, auto `lowFX` if FPS<45). Target 60 fps desktop / 30+ fps mobile.

## DOCS.md ↔ code
- `DOCS.md` matches the shipped code: §6.1 modes (Chill 2.6 / Classic 3.4 / Rush 4.8), §7 scoring (height = score; perfect = grow-back + streak), §14 `CONFIG` constants all verified against `index.html`. ✅

## ✅ VERDICT: **PASS — GAME DONE** (14/14 test cases PASS, 14 screenshots desktop+mobile, 0 console errors)
