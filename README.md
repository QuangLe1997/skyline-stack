# 🏙️ SKYLINE STACK — One-Tap 3D Tower Stacker

A one-tap 3D tower stacker. A block slides above your tower — tap to drop it, and whatever hangs over the edge is **sliced off** and tumbles away. Nail a near-perfect drop and the block doesn't shrink, it **grows back** — the pitch ladder climbs and the skyline shifts from cool cyan to **white-hot coral**. It speeds up as you climb. Miss the tower and it's over. Higher = better.

**▶ Live:** https://quangle1997.github.io/skyline-stack/ · part of [QUANG ARCADE](https://quangle1997.github.io/arcade/)

## Features
- **Core loop:** slide → tap-drop → slice the overhang; the block alternates X/Z slide axis each level.
- **PERFECT hook:** near-exact drops skip the trim, grow the block back, step the music pitch up one note, and turn the colour gradient hotter — chain a streak for a white-hot skyline.
- **3D & feel:** Three.js scene, smooth camera pan up the tower, sliced pieces fall as fading debris, subtle bloom, synthwave palette.
- **Modes:** Chill / Classic / Rush start-speed select. Endless height = the progression.
- **Juice:** slice particles + SFX, rising pitch ladder, combo popups, screen-color heat shift on streaks, near-fall wobble, haptics, calm synthwave loop.
- **Controls:** tap / click / **Space** to drop · **Esc**/**P** to pause.
- **Systems:** `localStorage` best height + longest perfect streak, mute + per-channel audio toggles, first-run How-to-Play, CDN/WebGL fail guard, auto quality drop on weak devices.
- Single `index.html`, zero build, mobile-first portrait, deployed on GitHub Pages.

## Run locally
```bash
python -m http.server 8794
# open http://localhost:8794
```
(ES modules / importmap need an HTTP origin — won't run from `file://`.)

## Docs
Full technical reference (level model, difficulty curve, scoring, balance constants, how-to recipes): [DOCS.md](DOCS.md).

---
Built by [QuangLe1997](https://github.com/QuangLe1997) · crafted with ♥ & Claude Code.
