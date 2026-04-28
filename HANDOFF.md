# Practice Room Diagram — Claude Code Handoff

## Project goal

A polished, shareable HTML/CSS top-down diagram of my band's practice room. Shows where each player sits, where amps and monitors are, and **most importantly the direction each speaker fires**. I'll be sharing this with the band so it needs to look good, not just functional.

Current state: working but positions and sound cone angles still need iteration. Doing this in chat is too slow — handing off so I can rapidly tweak positions in a real editor with hot reload.

## Files

- `practice-room-diagram.html` — single-file HTML/CSS/SVG diagram (the artifact to iterate on)

No build step. Just open the HTML file in a browser. Live reload via VS Code Live Server, `python -m http.server`, or whatever you like.

## The room (physical reality)

- Roughly 12 ft × 15 ft, slanted ceilings
- L-shaped, not a clean rectangle. Stairs alcove notches into the top wall, "bed" zone is in the bottom-right
- Hard parallel walls = muddy as hell, which is the whole reason this diagram exists

## The band

| Symbol | Role | Notes |
|--------|------|-------|
| V | Vocals | Lead singer, near stairs (top) |
| B | Bass (me) | I also sing backup. Sit at top-left of drum kit, near hi-hat/crash |
| G1 | Guitar 1 | Same latitude as V (top of room), against left wall |
| G2 | Guitar 2 | Mid-room, right side, against right wall |
| D | Drummer | Acoustic kit, bottom-middle. Newer drummer, plays loud |

## Gear positions (current)

| Item | Location | Faces / fires toward |
|------|----------|---------------------|
| G1 amp | Top-left wall | South (down into room) |
| G2 amp | Right wall, mid-room | West (left, across room) |
| Bass amp | Left wall, same latitude as drummer | East (right, toward drums) |
| M1 monitor | South of drummer | North, **fires at drummer** |
| M2 monitor | Against left wall, between G1 and bass | East, **fires right** — hits singer's right ear and bass's left ear |
| M3 monitor | Just left of G2 | East, **fires straight into G2** |

## Aesthetic direction (already established — keep this)

- **Graph paper / engineering blueprint vibe.** Warm cream paper background with subtle grid, drop shadows, hand-drawn warmth without losing precision.
- **Typography:** Fraunces (display, warm serif) + JetBrains Mono (technical labels). Don't swap these out.
- **Color palette** (already in CSS variables, don't drift):
  - `--paper: #f4ead5` — background
  - `--ink: #1a1814` — primary
  - `--accent-red: #b8412e` — vocals + amp sound cones
  - `--accent-blue: #2c5f7c` — guitars + monitor sound cones
  - `--accent-purple: #6b4a7a` — bass
  - `--accent-green: #4a6b3a` — drummer
- **Sound cones** are the centerpiece. Red gradient cones for amp output, blue gradient cones for monitor output. They should clearly communicate direction at a glance.
- "CURRENT STATE" stamp in top-right corner — leave it, useful for the eventual "PROPOSED" companion diagram.

## What still needs work

These are the things I want to iterate on faster:

1. **Cone angles and lengths.** Some cones look right, others don't quite line up with where the speaker is actually pointing. Each cone is an SVG `<g>` with a `transform="translate(x y) rotate(deg)"` — the rotation angle is what controls firing direction (0 = east, 90 = south, 180 = west, -90/270 = north).
2. **Position fine-tuning.** Player and gear positions are set as `top` / `left` percentages on absolutely-positioned elements. Look for the `/* ============ POSITIONING ============ */` block in the CSS — everything's grouped there so it's easy to nudge.
3. **L-shape geometry.** Right now the room is just a rectangle with a "bed" label and a dashed line hinting at the alcove. If we want to actually draw the L-shape with proper walls, that's a bigger refactor — probably switch the room outline from a CSS-bordered `<div>` to an SVG path.
4. **Monitor wedge rotation.** The wedge icons themselves get rotated via CSS `transform` to visually point in their firing direction. Cone direction (in the SVG) and wedge rotation (in CSS) need to match — easy to forget to update both when moving a monitor.

## Coordinate system gotcha

There are two coordinate systems that need to stay in sync:

- **CSS positioning** uses `top` / `left` as percentages of the room div (room aspect ratio is 12:16, so it's taller than wide).
- **SVG sound cones** use a `viewBox="0 0 100 133.33"` matching the same 12:16 aspect ratio, so coordinates roughly match the CSS percentages. A CSS `left: 60%` corresponds to SVG `x=60`, and CSS `top: 70%` corresponds to SVG `y≈70` (since 70% of 133.33 ≈ 93, but we're using the 100×133.33 box as if it scales to the percentages — `y` values in SVG are roughly 1.33× the CSS top%. Worth double-checking when something's off.)

Actually this is one of the things I'd love help cleaning up — having to think about two coordinate systems is annoying. Maybe move everything into a single SVG?

## Possible refactor ideas (open to your judgment)

- **Single SVG approach:** Draw the entire diagram (room outline including L-shape, people, amps, monitors, cones) as one SVG. Eliminates the dual coordinate system and lets the L-shape be drawn properly with a path. Tradeoff: lose some of the CSS shadow/gradient richness on the gear icons, but probably worth it.
- **Config-driven layout:** Pull all positions into a JS object at the top of the file (`const layout = { vocals: {x: 50, y: 22}, ... }`) so iteration is one-place editing instead of hunting through CSS.
- **Direction as vectors not rotations:** Instead of `rotate(90)` etc., specify "fires from (x1,y1) to (x2,y2)" and compute the cone. More intuitive when matching to a player position.

Any of those would make iteration much faster. Use your judgment on which to do — or none, if the current structure is fine for quick tweaks.

## Out of scope (don't do these)

- Don't add a "proposed layout" version yet. This is just the current state.
- Don't add interactivity, animations, or JS unless it's genuinely useful for iteration speed.
- Don't change the aesthetic direction (fonts, colors, paper background) — that's locked.

## How I'll know it's done

When I can show this to the band and they immediately understand:
1. Where everyone sits
2. Where every amp and monitor fires
3. That this room has a sound problem worth solving

Right now it's close but the cone angles and a couple of positions feel slightly off.
