# Workthrough — Vane House (horror metroidvania)

**Date:** 2026-08-15
**Task:** "Make a horror metroidvania investigative game in HTML where the user avoids ghosts, ghouls, or other horrors while finding the secret of the mansion that they arrive at the start of the game."
**Deliverable:** `vane-house.html` (~75 KB, single file, zero dependencies) + `README.md`

---

## 1. Approach

Single HTML file, vanilla JS + canvas, no build step, no assets. All audio is procedural (WebAudio oscillators/noise). The game was built in four phases:

1. **Design** — narrative, map layout, ability/item gating, enemy AI types, win condition.
2. **Authoring** — one `write` of the complete file (~1,800 lines).
3. **Verification** — a map validator and a 57-assertion headless gameplay simulation (game logic executed in Node with DOM/canvas/WebAudio stubs).
4. **Visual confirmation** — real-browser screenshots via headless Chrome, analyzed with a vision model.

The core insight that shaped everything: the "investigative" requirement and the "metroidvania" requirement are the same loop — **documents grant knowledge, knowledge unlocks geometry, geometry unlocks items, items unlock deeper geometry**, culminating in a ritual/boss that pays off the whole investigation.

---

## 2. Design decisions

### Narrative / the secret
- **Setting:** Vane House, October 1912. Player is a private investigator; the Vane family (Aldous, Eleanor, Margaret, Thomas) vanished in a single night in 1911.
- **The secret:** Aldous Vane performed a binding rite to cheat death, anchoring his family's souls to the house. They are still inside it.
- **The undoing (win condition):** three tokens on the circle (locket, bone flute, signet ring), the master's gaze pinned **three times** by the silver mirror, then his name spoken at the altar in the Sanctum.
- **10 collectible documents** carry the story: newspaper clipping, final diary entry, unsent letter (hints the portrait is a door), housekeeper's note (hints the cellar), list of names, a child's chalk drawing, a letter in the hall (warns the mirror must hold *three times*), Eleanor's journal (names the three tokens), a gravestone in the crypt, and the transcribed rite (the full ritual recipe). Finding all 10 auto-assembles a "The Truth" entry in the journal.

### Metroidvania gating (progression chain)
```
start (front door slams shut behind you at t≈3.4s)
├─ kitchen ──────────── pale candle (light radius up; repels shades)
├─ dining room ──────── brass key
│   └─ (2F) nursery (locked, brass) ── silver mirror + locket (token 1)
├─ 2F ladder → attic ──── bone flute (token 2)
├─ top-hall portrait door (hidden) → study
│   └─ iron key + rest point
└─ cellar door (locked, iron) → cellar
    ├─ crypt (gouls) ──── signet ring (token 3), gravestone, rite text
    └─ sanctum ────────── altar / final boss
```
- Floors: ground (40×28), second (40×28), cellar (40×28), attic (24×12).
- Rest points (sofa, study chair, bed, cellar cot) = sanity heal + checkpoint + ghost reset.

### Enemy AI
| Type | Floors | Behavior |
|---|---|---|
| **Wraith** | ground, 2F | patrols waypoints; sees player via grid line-of-sight **and a facing cone** (1.2 rad); chases; loses interest after 3.5 s of no sight |
| **Ghoul** | cellar | omnidirectional-ish (wide sense), faster patrol (52) and chase (105) |
| **Shade** | attic, cellar | no line-of-sight; blink-teleports to a random point 70–130 px around the **player** every ~6.5 s when the candle is off; candle light pushes it back |
| **The master's shade** (boss) | sanctum | spawns when the rite begins; the sanctum door seals; chases (100→118 px/s as enrage); each mirror hit pins it (+3.2 s stun, knockback); 3 pins → kneels → name can be spoken |

### Player systems
- **Sanity:** 3 candles; a touch costs 1 + knockback + 1.8 s invulnerability; 0 = death → fade → respawn at last rest point (sanity refilled, all ghosts reset to home, rite progress discarded with "the circle is broken").
- **Stealth:** holding SHIFT while moving = hold breath → detection range ×0.55 (wraiths) / ×0.78 chase speed.
- **Candle (C):** light radius 112 → 180 px; shades are suppressed near lit candle; flicker noise on the radius.
- **Silver mirror (F):** 9 s cooldown; 125 px pulse repels normal ghosts (7 s, knockback) or pins the boss.
- **Journal (J):** objective line, inventory, all found clues; reading a note opens a popup and pauses the world.

### Rendering / atmosphere
- Darkness is an offscreen canvas: near-opaque fill + `destination-out` radial hole at the player (plus small glow holes for rests, altar, stairs, and uncollected items).
- Ghosts are drawn **on top of** the darkness with distance-modulated alpha (≈0.14 faint in the dark → 1.0 in the light), so they read as shapes in the dark and details in the light.
- Per-floor palettes (wood / wood / stone / attic planks), rugs, black-water pools with ripples, portrait whose eyes track the player within 240 px, fog-of-war minimap (2 px/tile, explored tiles only), sanity candles, hotbar with mirror cooldown arc, objective tracker, message toasts.
- Film grain (192×120 noise canvas refreshed at ~10 Hz, random jitter), CSS vignette, screen shake, white flash (mirror) / red flash (hurt).
- Procedural audio: 51/53.7 Hz drone + LFO, filtered wind noise, footstep noise bursts, state-scaled heartbeat (2.4 s idle → 0.62 s chased), dissonant stinger on alert, door creaks, mirror chime, shade whisper, boss roar, chord for the final rite. `M` mutes.

---

## 3. Map authoring

Maps are ASCII grids (one char = 32 px tile):
`#` wall · `.` floor · `D` door · `L` brass-locked · `I` iron-locked · `F` front door · `P` portrait door · `u`/`d` stairs · `l` ladder · `A` altar · `~` black water (solid) · `B` rest (solid).

All notes, items, rest points, ghost spawns and patrol waypoints are data-listed with tile coordinates, so the validator can prove they sit on floor tiles and are reachable.

Layout (ground floor): central hall; library (left, door at 13,6); kitchen (left, door 13,19); dining room (right, door 26,6); chapel (right, door 26,19); hidden study top-left behind portrait (17,4) with stair-room top-right (grand stairs at 23,2); cellar door room bottom-center behind iron door (19,23) with stairs (19,25); front door (15,27).

---

## 4. Build & verification log

### 4.1 Map validator (Node)
Checks: uniform row lengths per floor; border walls; stair tiles + landing targets; every note/item/rest/ghost spawn/patrol point on floor; expected door chars at expected coords; **multi-floor BFS reachability** from the start tile (doors treated as open) proving every objective is obtainable.

**First run — 12 failures, all real bugs caught:**
1. Ground floor rows 6, 19, 26 were **41 chars** — the door column duplicated instead of replacing the wall column, shifting doors one column right. Library/kitchen were sealed (diary note, housekeeper note, and the candle were unreachable).
2. Second floor row 8 the same bug → nursery door vanished (brass key + mirror + locket unreachable), master's bed unreachable.
3. Cellar row 10 was 41 chars → the `u` stair char missing (basement landing wrong).
4. Consequence: floor 3 (attic) and several objectives unreachable in BFS.
5. Row 26 additionally had the cellar room's bottom wall mis-sealed, which would have let players **bypass the iron door** and reach the cellar stairs unkeyed.

Fixes: rewrote the five rows (`D` replacing `#` at the wall column; bottom wall `#####` fully sealing the cellar room; 18 dots + `u` + 19 dots for the cellar stair row) and removed a dead "self-healing row" hack left in the data block.

**Second run:** all 4 floors validated (40×28, 40×28, 40×28, 24×12), reachability 893 / 938 / 936 / 215 tiles, every objective reachable, ghost patrols reachable from spawns. **ALL CHECKS PASSED.**

### 4.2 Syntax
`node --check` on the extracted script. First run failed on a **malformed `rugs` array literal** I had authored (unbalanced nested arrays in the render pass). Replaced with a clean per-floor object of rect rugs. Re-check: clean.

### 4.3 Headless gameplay simulation (Node + stubs)
DOM (`getElementById`, `classList`), canvas 2D (Proxy no-ops with real `measureText`/gradients), WebAudio (fake graph), and `requestAnimationFrame` stubbed; the game script is eval'd, then driven by synthetic keyboard events. 57 assertions:

title → Enter → intro → play · WASD movement · note pickup (popup pauses world, journal entry added) · candle pickup + C toggle · stairs ground↔2F · ladder 2F↔attic · A* (open hall path, blocked by closed door, path through opened door) · LOS (false through wall, true in hall) · wraith deterministic chase (facing cone) · wraith approaches without phasing through walls · wraith gives up → patrol · mirror sets cooldown + repels wraith · ghoul touch → death → Enter → respawn at rest with sanity 3 · ghosts reset on respawn · rest heals + refuses while chased · portrait door opens · nursery/iron doors locked without keys, open with keys · cellar stair · brass key / mirror / ring pickups · objective updates through all milestones · altar rite starts (boss active, sanctum door seals) · boss pinned 1/2/3 with 80 px standoff · "speak the name" objective · win sequence → win screen → Enter → back to title.

**First run — 22 failures.** Triage:
- **Test-harness bugs** (not game bugs): `itarget` exported by value (snapshot `null` — fixed with a `getitarget()` closure); intro wait too short (46 chars/s needs ~8.9 s); test standing the player on top of enemies (instant death cascades); non-adjacent portrait test position; a division by the export object instead of the tile constant (`wa.x/T` where `T` was the handle, not 32).
- **3 real game bugs found:**
  1. **Ghost wall-phasing**: chase/patrol code had a "walk straight at target" fallback when A* returned nothing; with no pathing the ghost walked through walls (and would corner the player through them). Fixed: direct-target only allowed when `dist < 30` **and** clear line-of-sight; unreachable patrol waypoints now wait 1.5 s and skip to the next waypoint instead.
  2. **Shade teleported relative to itself** (a random jump up to 150 px from the shade), which often moved it *away* from the player — wrong for the "it appears behind you" horror beat. Fixed: teleport target is a random point 70–130 px around the **player**, validated as walkable.
  3. **Boss direct-target** had the same unguarded wall-phasing fallback — now line-of-sight gated.

**Final run: 57/57 SMOKE TEST PASSED** (plus a NaN-hunt: one intermediate repro suggested a ghost position NaN; root cause was the harness's `T` vs `Tc` mistake, not the game — verified with an instrumented 200-frame trace showing the wraith on walkable tiles the whole time).

### 4.4 Visual confirmation (headless Chrome)
Screenshots at `file://` with `--headless=new --virtual-time-budget`, each under a 60 s `timeout` (the first un-timed attempt hung on the endless rAF loop — per user instruction, all test runs now carry hard timeouts).
- Title screen: clean, legible, controls listed, no glitches.
- Gameplay frame (driver script auto-starts the game, grants candle + mirror, places the player in the hall with a chasing wraith): confirmed the warm circular candlelight, the player figure, the pale translucent wraith with red eyes and trailing tail, the 3 sanity candles + floor label, the fog-of-war minimap, both hotbar slots (C candle / F mirror with cooldown), the "The front door slams shut behind you" toast, the objective line, and a faint item glow in the darkness. Grain + vignette present. No rendering glitches.

(First driver attempt produced the title screen twice because the auto-play script assumed the shipped file exports a game handle — it doesn't; fixed by appending an in-scope `globalThis.__T = {G, P, ghosts, T}` export only in the throwaway driver copy, never in the shipped file.)

---

## 5. Deliverables

| File | Purpose |
|---|---|
| `vane-house.html` | The complete game (maps, engine, AI, audio, UI, overlays) |
| `README.md` | Controls, progression chain, enemy table, the secret |

**Final state:** all four verification gates green — map reachability, syntax, 57/57 gameplay assertions, and in-browser visual checks.

## 6. Notes / possible next steps
- Mobile/touch controls are out of scope (keyboard game).
- The "rest to reset the boss" escape hatch is deliberate checkpoint design; if it feels too easy, the fix is one line in `anyChasing()`.
- Easy content additions (all data-driven): more notes, extra patrol routes, a fourth token, a second boss phase, or a hidden room behind another portrait.
