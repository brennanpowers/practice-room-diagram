# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static three-page site documenting a band practice room layout. Hosted on GitHub Pages at `brennanpowers.github.io/practice-room-diagram/`. The main page is an interactive top-down room diagram that compares "current" and "proposed" gear placements (with a hidden "ideal" scenario reachable only by URL).

## Build / dev / deploy

There is **no build step, no package manager, no tests, no linter.** Every page is a single self-contained HTML file with inline CSS and inline JavaScript.

To work on it:
- Edit the HTML file directly
- Open it in a browser to preview (no dev server required, just `file://` works)
- For URL-parameter testing, use a local static server (e.g. `python -m http.server`) so `?layout=...` query params resolve

To deploy:
```sh
git add -A && git commit -m "..." && git push
```
Pages auto-rebuilds in ~30s. The `main` branch is served at root.

## Files

- `index.html` — main diagram (SVG room rendered from a JS config object). All scenario data and rendering code lives here.
- `references.html` — citations page documenting the acoustic principles behind the proposed layout
- `eq.html` — EQ tuning guide for the band's gear (linked from references, not from main)
- `HANDOFF.md` — original project context written before the diagram existed. Useful background on the room and band; treat as historical reference, not current state
- `original_sketch.png` — hand-drawn source sketch the diagram was built from

Cross-page links:
- main footer → references
- references header → eq + back to main
- eq header → references + back to main

## index.html architecture

The page renders an SVG diagram of the room from a config object. Read these in order to understand the system:

1. **`ROOM_GEOMETRY`** — fixed room polygon + bed alcove + stairs alcove, shared across all scenarios.
2. **`SCENARIOS`** — keyed by layout name (`current`, `proposed`, `ideal`). Each scenario has `MICS`, `AMPS`, `MONITORS`, `DRUM_KIT`, and `MIXERS` arrays. The full layout (positions, cone aim, labels) lives entirely in this object — the rendering code is generic.
3. **`ACTIVE_KEY`** — read from `?layout=...` URL param, defaults to `current`. Selects which scenario object drives the render.
4. **Render functions** — `drawCone`, `drawAmp`, `drawMonitor`, `drawMic`, `drawMixer`, `drawDrumKit`. Each iterates the corresponding array from the active scenario.

To **add a new scenario**: append a key to `SCENARIOS` with the same shape as `proposed`. Surface it in the nav by adding an `<a data-key="name">` to `.layout-nav` in the header. Leave it out of the nav to make it URL-only (this is what `ideal` does).

### Coordinate system convention

- Origin at top-left, +x east, +y south (standard SVG screen coords).
- `fire` angle on amps and monitors uses **clockwise-from-east** (NOT standard math convention): `0° = east, 90° = south, 180° = west, 270° = north`. This matches SVG `rotate()` directly.
- Coordinates are **relative** — they don't represent real-world feet/inches. The legend explicitly states this. Numeric axis rulers exist only as a reference vocabulary so positions can be discussed precisely (e.g. "move to (500, 800)").

### Scenario-aware legend

Legend `<div class="legend-section">` elements may carry a `data-scenario` attribute with a comma-separated list (e.g. `data-scenario="proposed,ideal"`). The render code shows/hides these based on whether `ACTIVE_KEY` is in the list. Sections without `data-scenario` always render.

When adding a new scenario that should reuse existing legend content, add the scenario name to the relevant `data-scenario` lists rather than duplicating the HTML.

### Element schemas

Inside a scenario:

```js
MICS:    [{ id, x, y, face, label, tagAt }]
AMPS:    [{ id, x, y, fire, length, spread, tagAt, tilted? }]
MONITORS:[{ id, x, y, fire, length, spread, tag, tagAt, optional? }]
DRUM_KIT:{ x, y, size }
MIXERS:  [{ x, y, label, passive? }]
```

- `tilted: true` on amps shows an amber wedge under the body (suggests amp on a case/stand) and appends `↑` to the tag
- `optional: true` on monitors renders the body and cone with a dashed style and appends `(opt)` to the tag
- `passive: true` on mixers renders a smaller plain box (no knobs)
- `face` on mics draws a dashed direction arrow showing where the singer is looking

## Aesthetic / style conventions

The look is intentional — graph-paper / engineering-blueprint vibe. Don't drift:

- **Color palette** (CSS variables in each file): `--paper: #f4ead5`, `--ink: #1a1814`, `--accent-red: #b8412e` (vocals + amp cones), `--accent-blue: #2c5f7c` (monitors + speaker cones), `--accent-amber: #b8862e` (tilt indicator + boundary)
- **Type**: Fraunces (warm serif body) + JetBrains Mono (technical labels). Both loaded from Google Fonts.
- **Mobile**: each page has a `≤540px` breakpoint that stacks the header and shrinks chrome. Touch targets in the layout nav are padded.

## Domain context worth knowing

The proposed layout makes specific acoustic claims (cardioid mic null geometry, parallel-wall flutter mitigation, M200 Master+Monitor mode routing, etc.). These are documented with citations in `references.html` and operationalized in `eq.html`. When making changes that touch acoustic decisions, check whether the references or EQ pages need a corresponding update — they are intended to stay in sync with the diagram's claims.

Memory at `~/.claude/projects/-Users-brennan-Documents-PracticeSpaceRedesign/memory/` contains user/project context that persists across sessions (e.g. mic-to-person mapping). Read `MEMORY.md` there before making domain assumptions about the band.
