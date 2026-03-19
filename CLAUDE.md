# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build system, server, or dependencies to install. Open any HTML file directly in a browser:

```
open -a "Google Chrome" snapback-collection.html
open -a "Google Chrome" index.html
```

All CSS, HTML, and JS live in single self-contained files. CDNs are loaded from the internet — a network connection is required.

## Files

| File | Purpose |
|---|---|
| `snapback-collection.html` | Fan-facing card collecting app (primary focus) |
| `index.html` | B2B event planning / ROI dashboard |
| `snapback_event_data_final.html` | Partial HTML snippet (table + styles only, no doctype) — embedded into a parent context that provides CSS variables |

## snapback-collection.html Architecture

Single-file app: all CSS in `<style>`, all data + logic in one `<script>` block at the bottom, HTML screens inline in `<body>`.

**CDNs loaded (in order):**
- GSAP 3.12.5 — all animations (`gsap.to`, `gsap.fromTo`, `gsap.timeline`)
- VanillaTilt 1.8.1 — 3D tilt on `.card` elements via `initTilt()`
- canvas-confetti 1.9.2 — `confetti()` calls in legendary rarity FX and end screen
- tsParticles 3.3.0 — ambient particle field on pack opening overlay

**State:**
- `S` — global app state: `{ coins, selectedPack, selectedNFLTeam, selectedWCCountry, screen }`
- `PO` — pack opening state: `{ phase, pack, cards, flipped, legDelay, revealing }`

**Navigation:** `go(screenName)` swaps active screen and triggers GSAP directional slide. Screen IDs map to `screen-{name}` elements. `SCREEN_DEPTH` array determines slide direction (forward/back).

**Pack opening flow:**
1. `startOpen()` — opens overlay, GSAP pack entrance, tsParticles ambient field
2. `onTapPack()` — explosion FX (shockwave + shards), builds `PO.cards` from `buildPool()`, calls `renderCardsPhase()`
3. `renderCardsPhase()` → `flyCardIn(i)` — cards fly in one by one with screen shake
4. `tapRevealCard(i)` — mandatory tension pulse → GSAP flip → `triggerRarityFX(rarity, cardEl)`
5. `triggerRarityFX()` — common/rare/epic/legendary each have their own FX sequence; legendary sets `PO.legDelay = 5500` to delay end screen
6. `showEndScreen()` — emotional message based on best rarity pulled, optional confetti

**Data:** All card/team/pack data is hardcoded as `const` arrays at the top of the script (`snapbackCards`, `nflTeams`, `wcCountries`, `packDefs`). No API calls.

**Render pattern:** `renderSnapback()`, `renderNFLTeam()`, `renderWCCountry()` write innerHTML directly then call `initTilt()` + `cascadeCards()`. `renderPackList()` rebuilds the pack selection list and resets the open button state.

**Key CSS patterns:**
- Rarity colors via CSS vars: `--common`, `--rare`, `--epic`, `--gold`
- Pack overlay uses `#pack-overlay.on` (toggled class) + `display:flex`
- Screen transitions: `.screen.active` = `display:block`; GSAP handles opacity/x
- `#po-fx` is an absolutely-positioned full-overlay div used as a container for all mid-animation DOM FX elements (shockwave, shards, embers, bolts, etc.) — always cleared with `fx.innerHTML=''` when done

## index.html Architecture

React 18 via CDN (no JSX compiled at build time — Babel standalone transforms at runtime in the browser). All component logic is in a single `<script type="text/babel">` block. Event data is hardcoded inline. No state management library.
