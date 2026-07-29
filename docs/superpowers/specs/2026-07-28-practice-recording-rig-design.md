# Practice Recording Rig — Design

**Date:** 2026-07-28
**Status:** Implemented in `e30c1eb`. Hardware not yet purchased; four items remain to verify
in-room (see Open questions).
**Scope:** Add a portable recording rig to the `proposed` layout in `index.html`, plus supporting documentation in `references.html`.

### Implementation note not anticipated by this spec

The signal-chain legend panel carries `data-scenario="proposed,ideal"` — it is **shared with the
hidden `ideal` scenario**, which has no `REC` mic. Adding the recorder block to that SVG therefore
made `ideal` claim a recorder it does not have. Fixed by wrapping the block in
`<g data-scenario="proposed">`: the existing show/hide runs `querySelectorAll('[data-scenario]')`
and sets `style.display`, which works on SVG groups as well as legend divs. Costs a little unused
whitespace at the bottom of `ideal`'s panel, since the viewBox stays tall.

If `ideal` should eventually gain a recorder, its aim needs re-deriving — that layout moves the
mics to the north end, so the 125° figure derived for `proposed` does not carry over.

---

## Problem

The band wants to record practice sessions. Past attempts used a single room mic and the
vocals were consistently too quiet in the resulting mix.

Analysis of the `proposed` layout shows this is not a placement problem. The mains sit at
the south wall (y ≈ 1370) and the drum kit sits at (800, 1158), so every position that moves
closer to the mains also moves closer to the kit. Across four candidate positions the best
mains-to-drums ratio was **−1.7 dB** — the drums win everywhere in this room. Moving the mic
buys a couple of dB, not the 10+ dB the problem calls for.

The fix is therefore structural, not positional: **the lead vocal must arrive on its own
track**, so its level is set after practice rather than fought for during it.

---

## Constraints

1. **The room rig is semi-permanent; the recording rig is temporary and travels.**
   The practice room must work fully with the recording equipment absent. Nothing may be
   left behind, and no room-side gear may depend on the recording rig being present.
2. The M200-BT is a **powered** mixer and contains the amplifier driving the passive mains.
   It cannot be removed from the live path.
3. Minimum practical setup and teardown time.

---

## Decision

**Zoom H4essential as a standalone 4-track recorder.** No L-8, no external phantom, no
mixer changes.

| Track | Source | Format |
|-------|--------|--------|
| `TrMic` (stereo) | Built-in X/Y pair — room: drums, amps, ambience | 32-bit float |
| `Tr1` | MIC 1 (lead vocal), passive split | 32-bit float |
| `Tr2` | MIC 2 (backup vocal), passive split | 32-bit float |
| `TrLR` | Stereo mix of the above, produced automatically | 32-bit float |

### Why this configuration

- **32-bit float on every track, vocals included.** A mis-set level is recoverable in post
  rather than a ruined take. No one has to act as engineer during practice.
- **No phantom power anywhere in the design.** The vocal mics are dynamics and the X/Y pair
  is internal. This is what makes plain passive Y-cables acceptable — there is no DC near
  any split junction. Every L-8-based alternative reintroduces phantom onto the split
  junctions, because the L-8's 48 V is switched globally across all six mic inputs.
- **Two split junctions instead of four**, halving impedance-loading exposure.
- **Smallest possible kit**: one recorder and two Y-cables. No mic stand — it sits on the
  standing desk already at (600, 900).
- The L-8 stays home and remains available as a documented upgrade.

### Accepted limitation

MIC 3 (G2 vocal) and MIC 4 (drummer vocal) are **not** isolated. They appear only within the
stereo room track. This is a deliberate trade: they are backing vocals, and isolating them
would require either the L-8 (bigger kit, 24-bit, phantom complications) or an H6essential
(replacing a mixer already owned).

---

## Bill of materials

| Item | Qty | Approx. | Notes |
|------|-----|---------|-------|
| Zoom H4essential | 1 | $199 | |
| XLR passive Y-splitter cable | 2 | ~$15 ea | MIC 1 and MIC 2 |
| microSD card | 1 | ~$15 | |
| Small desk stand or foam isolation pad | 1 | ~$15 | Lifts capsules off the desk surface |
| **Total** | | **~$260** | |

No mic stand required — the recorder sits on the standing desk at (600, 900). Batteries:
2×AA, ~10 h runtime; USB-C bus power also supported. XLR runs from MIC 1 (680, 600) and
MIC 2 (400, 1000) to the desk are both short.

---

## Signal chain

```
ROOM SIDE — unchanged, functions with the rig absent
  MIC 1 ──► Y ──┬──► M200 CH1 ──┐
                └──► SRD210 wedge│
  MIC 2 ──► Y ──┬──► M200 CH2 ──┤
                │                ├──► M200 ──► MAIN L / MAIN R
  MIC 3 ────────┼──► M200 CH3 ──┤
  MIC 4 ────────┼──► M200 CH4 ──┘
                │
SIDECAR — travels
                └──► (second leg of each Y)
  MIC 1 leg ──────► H4essential INPUT 1     → Tr1
  MIC 2 leg ──────► H4essential INPUT 2     → Tr2
  built-in X/Y ───► H4essential MIC          → TrMic
                    └──► microSD  +  USB-C ──► MacBook / GarageBand
```

Teardown: unplug two Y-splitters, restore the direct mic cables. Room returns to its
documented state with nothing left behind.

**MIC 1 becomes a three-way split** (wedge, M200, H4essential). This is the one remaining
loading risk. It is testable: if the lead vocal returns thin or quiet, a single transformer
splitter on that channel resolves it. Do not pre-buy — MIC 1 has already run a two-way
passive split successfully.

---

## Recorder configuration

- **Recording mode:** all three channel buttons armed (`MIC`, `1`, `2`)
- **Phantom:** OFF on both inputs (per-input setting; vocal mics are dynamic)
- **Lo Cut:** ON for Input 1 and Input 2 — the recording-side equivalent of the M200's
  channel low-cut, mitigating bass-amp bleed into the vocal mics. The M200's own low-cut
  continues to serve the live sound independently.
- **Sample rate:** 48 kHz
- **Gain:** set once via the MIXER button, roughly. Precision is unnecessary given 32-bit float.

Levels do not need monitoring during a session — 32-bit float makes a wrong setting
recoverable rather than fatal. The desk position keeps the recorder within reach regardless,
which is the main reason it was chosen.

---

## Placement

**Position: (600, 900), ~4 ft height** — on the standing desk already in that spot.
Chosen for operational convenience over acoustic optimum, deliberately: a rig that is
trivial to set up gets used, and the recorder must be reachable to arm and stop.

**Aim: 125°** (south-southwest), the bisector of the instrument spread.

| Source | Distance | Bearing | Off-axis @125° | Cardioid response | Net vs. drums |
|--------|---------:|--------:|---------------:|------------------:|--------------:|
| WEDGE | 80 | 270.0° | 144.8° | −20.8 dB | −8.6 dB |
| MIC 2 backup | 224 | 153.4° | 28.2° | −0.5 dB | +2.8 dB |
| G2 AMP | 320 | 20.1° | 105.1° | −8.6 dB | −8.5 dB |
| DRUMS | 326 | 52.2° | 73.0° | −3.8 dB | reference |
| B1 BASS | 570 | 148.3° | 23.0° | −0.4 dB | −5.2 dB |
| G1 AMP | 767 | 230.3° | 105.1° | −8.6 dB | −16.1 dB |

### Consequences, stated plainly

- **The desk sits inside the band.** The four instruments span a **210° arc** from here
  (G1 behind at 230°, G2 in front at 20°). No directional pair can cover that. Both guitars
  land ~105° off-axis regardless of aim.
- **G1 is the casualty**, landing ~16 dB below the drums on direct sound.
- **Drums arrive 8.4 dB louder** than they would at the north position (326 vs 862 units).
- **The wedge is only 80 units away.** This is the one genuinely close problem source: it
  carries the lead vocal, and heavy bleed into the room track erodes the independent vocal
  control this whole design exists to provide. Aiming at 125° places it **144.8° off-axis
  (−20.8 dB)**, which is why a directional pair is the right choice at this position.
  **Do not aim north.**

### Caveat on the figures above

These are direct-sound, free-field calculations. In a small untreated basement the critical
distance is only a few feet, so beyond that the reverberant field dominates and actual
balances come out substantially more even than this table implies. G1 will reach the mic via
reflections and will not be as absent as −16 dB suggests. Treat the table as showing the
*direction* of each trade, not its literal magnitude — and specifically as justification for
the aim, which is the one decision it genuinely determines.

### Physical notes

- **Lift the recorder off the desk surface** (small stand, foam pad, or place at the desk
  edge). A hard surface immediately under the capsules causes comb filtering from the
  reflection arriving a fraction of a millisecond after the direct sound.
- 4 ft is below the amp cabinets and near desk level; acceptable, not ideal.
- **Stereo orientation:** G1 and G2 sit on opposite sides of the field and will image hard
  left/right. Which is which depends on physical orientation; swap in GarageBand if reversed.

### If the room track proves too drum-heavy or too thin on G1

The north position **(500, 350), aim 120°, ~6 ft** is acoustically better: a 127° instrument
arc with all four sources within ~3 dB of each other, and G1 landing 4 dB *above* the drums.
It is documented here as the fallback. The reason it was not chosen is that it requires a
boom stand in the middle of the room and puts the recorder out of easy reach.

---

## Diagram changes — `index.html`

1. **Add `REC` to `proposed.MICS`:**
   ```js
   // Zoom H4essential on the standing desk, ~4 ft. X/Y stereo pair aimed 125° — the
   // bisector of the instrument spread, and critically AWAY from the WEDGE 80 units north
   // (144.8° off-axis, -20.8 dB) so lead-vocal bleed stays out of the room track.
   // Also records MIC 1 / MIC 2 splits on its two XLR inputs. Sidecar: travels with the kit.
   { id: 'REC', x: 600, y: 900, face: 125, label: 'REC · H4e', tagAt: 'east', stereo: true },
   ```

   Note `tagAt: 'east'` — at (600, 900) the label would otherwise collide with the WEDGE tag
   at (600, 820) and MIC 2 at (400, 1000). Verify visually and adjust.

2. **New `stereo: true` mic flag** in `drawMic`. The existing `omni: true` branch draws
   concentric rings for 360° pickup, which now misrepresents a directional X/Y pair. The
   `stereo` branch should use the amber accent (as `omni` does) but draw a **dashed amber arc
   or cone** along `face` indicating the capture field, replacing the concentric rings.
   Reuse `drawCone`'s geometry if practical.

3. **`keyboard` scenario:** its existing `REC` entry is at (500, 350) with `omni: true`,
   describing a B-5 that is no longer the chosen mic. Move it to (600, 900) with
   `face: 125, stereo: true` to match. The keyboard layout keeps the SRD210 wedge at
   (600, 820), so the same 80-unit proximity and the same aim-away-from-the-wedge reasoning
   apply unchanged. Its trailing comment about boom-stand capsule height (~7–8 ft) is now
   wrong and must be replaced.

4. **Legend:** extend the existing REC entry from `data-scenario="keyboard"` to
   `data-scenario="proposed,keyboard"`, and revise its text — it currently reads
   "omni for practice recording (Focusrite → laptop)", which is now wrong on both counts.
   Add a short recording-chain note covering the two vocal splits.

5. **Signal-flow SVG (proposed):** add the sidecar path — MIC 1 and MIC 2 splits to the
   recorder — rendered dashed to distinguish it from the permanent room chain.

---

## Documentation changes — `references.html`

Add a principle covering the recording chain:

- Why placement alone cannot solve the vocal-level problem in this room (the mains-versus-drums
  geometry, with the −1.7 dB best case)
- Why the M200 cannot provide isolated vocals: it exposes only STEREO OUT (summed) and
  MONITOR OUT (mono mix of all inputs). No direct outs, no channel inserts.
- Why passive splitting is acceptable *here specifically*: no phantom anywhere in the design.
  Note explicitly that this conclusion depends on the recorder's per-input phantom being off,
  and would not hold if a condenser were added to a shared preamp bus.
- The MIC 1 three-way split as a known risk with a defined remedy.

---

## Rejected alternatives

| Option | Why rejected |
|--------|--------------|
| **L-8 as front-end mixer**, M200 as power amp | Puts the travelling rig in the live path. Room would not function without it. Violates constraint 1. |
| **L-8 as parallel recorder**, 4 splits | Works, but 24-bit with six channels to gain-stage, and the L-8's *global* phantom forces DC onto every split junction if any condenser is used. Bigger kit. Retained as the upgrade path. |
| **B-5 omni + Focusrite Solo** | Mono room track only; Solo has one mic input. Superseded. |
| **Second Behringer B-5 as a stereo pair into L-8 ch 5/6** | Genuinely competitive: $73, 140/150 dB SPL, natively sample-locked, better room imaging. Rejected because it requires the L-8 to travel every session, needs two stands, is 24-bit, and reintroduces phantom (the L-8's global 48 V lands on the vocal splits). |
| **H6essential** ($299) | Would give all four vocals plus stereo room in one box, all 32-bit float. Rejected as it duplicates an already-owned L-8 for $100 more than the H4essential. |
| **Tascam DR-40XP** ($259) | Closest non-Zoom competitor. 5 dB less capsule headroom, $60 more. Its advantages (30 h battery, switchable A/B) do not apply here. |
| **Tascam DR-05XP / DR-07XP** | 2-track only, and their 3.5 mm input *replaces* the built-in mics. Cannot record room and vocals simultaneously. |
| **M200 Stereo Out → recorder** (summed vocals) | Simplest possible, but gives no per-singer control. Superseded by isolating MIC 1 and MIC 2 directly. |

---

## Variant — B-5 on a boom, recorder on the desk

Available at **zero additional cost** (the B-5 is already owned) and worth trying if the desk
room track proves unsatisfying. It exists because the H4essential's phantom is **per-input**,
so one channel can power a condenser while the other feeds a dynamic with phantom off.

| Track | Source |
|-------|--------|
| `TrMic` | Built-in X/Y at the desk (600, 900) — close, drum- and bass-forward |
| `Tr1` | **B-5, omni capsule**, boom at (500, 350) ~6 ft — the acoustically better room position. Phantom **ON**, Input 1 only |
| `Tr2` | MIC 1 (lead vocal) split. Phantom **OFF**, Input 2 |

The recorder stays on the desk where it is reachable, while an actual microphone occupies the
position the acoustics want — two room perspectives to blend, and the north position's even
instrument balance recovered.

**Cost:** MIC 2 loses its isolated track (it remains in both room tracks). Adds one boom stand
and one XLR run.

This variant is the reason the omni-versus-directional question does not have to be settled by
the desk position alone. The B-5's omni capsule has no off-axis loss, so it handles the 210°
instrument spread that defeats a directional pair — the exact weakness of the primary design.

---

## Upgrade path

If per-singer control of all four vocals becomes desirable, add the **already-owned L-8**:
split MIC 3 and MIC 4 as well, feed all four into L-8 ch 1–4, and keep the H4essential's X/Y
as the room pair (line out → L-8 ch 7/8 for sample lock, or its own card aligned by clap).
Nothing purchased for this design becomes redundant.

---

## Open questions

1. **M200 MONITOR OUT availability.** A considered variant puts MONITOR OUT (mono mix of all
   inputs) on Input 2 instead of MIC 2, trading discrete backup-vocal control for partial
   coverage of all four vocals. The layout runs the M200 in Master+Monitor speaker mode, and
   whether the line-level MONITOR OUT remains independently available in that mode is
   unverified. Check the Harbinger manual before considering this variant.
2. **H4essential X/Y capsule angle.** Whether it is fixed or switchable (as the H4n Pro's
   90°/120° is) was not confirmed. This affects stereo width, not coverage — cardioid
   capsules pick up well beyond the capsule angle — so it does not change the placement math.
3. **Wedge bleed into the room track — measure on the first session.** The wedge sits 80 units
   from the recorder. The aim places it 144.8° off-axis, but that figure is theoretical. Test:
   record with the wedge muted, then unmuted, and compare the room track. If lead vocal is
   still prominent in `TrMic` with the vocal fader down, the independent vocal control this
   design is built for is compromised, and the B-5 variant (which moves the room capture 550
   units away from the wedge) becomes the fix.
4. **MIC 1 three-way split.** Verify level and tone against the current two-way split on the
   first session. Remedy is a single transformer splitter on that channel; do not pre-buy.

---

## Verified specifications

| Claim | Source |
|-------|--------|
| H4essential: 4 simultaneous tracks; `MIC` / `1` / `2` independently armed; outputs `TrMic`, `Tr1`, `Tr2`, `TrLR` | H4essential operation manual |
| H4essential: per-input Phantom and Lo Cut (`Input 1` menu: Lo Cut / Phantom / 1&2 Link) | H4essential operation manual |
| H4essential: 130 dB SPL built-in X/Y; 3.5 mm line out; 4-in/2-out USB; 2×AA ≈ 10 h | zoomcorp.com H4essential product page |
| M200-BT: STEREO OUT (summed) and MONITOR OUT (mono mix) only; no direct outs or inserts | Harbinger M100-BT/M200-BT Owner's Manual |
| L-8: 48 V phantom switchable **globally** across all six mic inputs | Sound on Sound review; Zoom documentation |
| SRD210: MIC/LINE combo, GUITAR/MIC, AUX RCA, MIX OUT (XLR) | `equipment-manuals/600015_Manual_210222.pdf`, p. 4 |
| B-5: 140 dB SPL (150 dB with −10 dB pad); requires +48 V; cardioid + omni capsules | Behringer / Sweetwater specifications |

**Sources:**
[Zoom H4essential](https://zoomcorp.com/en/us/handheld-recorders/handheld-recorders/h4essential/) ·
[Harbinger M100-BT/M200-BT manual](https://harbingeraudio.com/wp-content/uploads/2023/09/Harbinger-M100BT-M200BT-Owner-Manual.pdf) ·
[Zoom LiveTrak L-8 review, Sound on Sound](https://www.soundonsound.com/reviews/zoom-livetrak-l-8) ·
[Tascam DR-40XP](https://tascam.com/us/product/dr-40xp)
