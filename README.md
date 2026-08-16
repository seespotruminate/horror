# Qwen3.8 (unsloth/Qwen3.8-27B-NVFP4) test

A test ran using the nvfp4 quant from unsloth and via vllm. Other this this line and the prior line, I didn't write any of this shit. This is just an example of some output. 


# VANE HOUSE

A single-file horror metroidvania / investigative game. Open `vane-house.html` in any modern browser. No build step, no dependencies — canvas + procedural WebAudio.

## Premise
October 1912. You arrive at Vane House to investigate a family that vanished in a single night four years ago. Read what you can, find what you need, and uncover the secret of the house — while the things inside it hunt you.

## Controls
| Key | Action |
|---|---|
| WASD / Arrows | Move |
| SHIFT (while moving) | Hold your breath — ghosts detect you from much less range |
| E | Interact (doors, notes, items, stairs, rest, altar) |
| C | Toggle your candle (bigger light; also pushes back shades) |
| F | Silver mirror — repels nearby ghosts; pins the master's gaze (long cooldown) |
| J | Journal (clues, inventory, objective) |
| ESC | Pause |
| M | Mute |

## The loop
- **Investigate**: 10 collectible documents (journal, notes, letters, a child's drawing) that piece together what happened. The journal assembles "The Truth" once all are found.
- **Metroidvania progression**:
  - The **pale candle** (kitchen) widens your light.
  - The **brass key** (dining room) opens the locked **nursery** (2F), where the **silver mirror** and first token hide.
  - A **portrait that is a door** (top hall) hides the study and the **iron key**.
  - The **iron key** opens the cellar door down to the crypt and the **Sanctum**.
  - Tokens: **locket** (nursery), **bone flute** (attic, via ladder), **signet ring** (crypt).
- **Horrors**:
  - **Wraiths** patrol and line-of-sight chase (facing cone; hold breath to blind them, mirror to banish).
  - **Ghouls** in the cellar — wider sense, faster.
  - **Shades** blink-teleport around you in the dark; candle light keeps them off.
  - **The master's shade** — final boss in the Sanctum: pin his gaze with the mirror **three times**, then speak the name at the heart.
- **Survive**: 3 sanity (candles). Touches drain one; at zero you're taken and wake at your last **rest point** (sofa, study chair, bed, cellar cot). Ghosts reset when you rest or respawn.

## The secret
The rite in the Sanctum: three tokens on the circle, the master's gaze pinned three times by silver, and his name spoken at the heart of the house.
