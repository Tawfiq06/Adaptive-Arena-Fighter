# Arena Fighter

A top-down 2D fighting game built entirely in C, running bare-metal on the **DE1-SoC** with a **Nios V/RISC-V** soft processor. No engine, no OS, no framework. Just direct hardware programming, a VGA framebuffer, and a game that actually feels good to play.

Two players fight across four hand-crafted arenas that physically transform mid-match. There is a full combat system, item pickups, custom name entry, and a trained reinforcement learning AI opponent.

> This repo contains documentation, screenshots, and technical writeups. Code is not posted per course policy.

---

## Quick Look

[![Demo Video](assets/thumbnail.png)](https://www.youtube.com/watch?v=YOUR_VIDEO_ID)
*Click to watch the demo*

| | |
|---|---|
| <img width="2030" height="1524" alt="greenreach" src="https://github.com/user-attachments/assets/e7e2e660-fb32-4ea4-a909-85b4bea9ee84" />| <img width="2030" height="1527" alt="coralpoint" src="https://github.com/user-attachments/assets/99d14ab8-d183-4fda-812a-a268f09d345a" /> |
| <img width="2030" height="1528" alt="tundra" src="https://github.com/user-attachments/assets/8a3bb045-0c5b-4920-80df-26106e51f419" /> | <img width="2030" height="1522" alt="ironridge" src="https://github.com/user-attachments/assets/0d9efd2c-48f7-40f1-89db-0671ae372d40" />|

---

## Table of Contents

1. [Gameplay](#gameplay)
2. [Arenas and Map Evolution](#arenas-and-map-evolution)
3. [The Storm](#the-storm)
4. [Combat System](#combat-system)
5. [Player Names and HUD](#player-names-and-hud)
6. [Audio System and Interrupts](#audio-system-and-interrupts)
7. [Decoration System](#decoration-system)
8. [Expandability](#expandability)
9. [Reinforcement Learning AI](#reinforcement-learning-ai)
10. [Performance Optimizations](#performance-optimizations)
11. [Hardware Overview](#hardware-overview)

---

## Gameplay

The game supports two modes: **Player vs. Player** (both players share one keyboard) and **Player vs. AI**. Before the match, players enter custom names, choose a game mode, then pick a map.

Each player has a light attack, heavy attack, ranged attack (bow), dash, and block. Health potions spawn periodically in the arena.

**Controls:**

| Action | Player 1 | Player 2 |
|---|---|---|
| Move | WASD | Arrow Keys |
| Light Attack | Q | M |
| Heavy Attack | E | K |
| Dash | Z | N |
| Block | X | J |

Navigate menus with Enter to confirm, Esc to go back.

---

## Arenas and Map Evolution

Each arena has its own evolution rule tied to its biome. Every ~1.5 seconds a tile transformation fires. The map slowly becomes a different place than where the match started, which keeps both players moving and thinking.

Tiles carry gameplay flags that the physics engine checks every frame: solid tiles block movement entirely, water tiles halve speed, lava and storm zones drain HP over time, and ice tiles preserve momentum so they cannot change direction until they hit something or reach the end of the ice.

---

### Greenreach

A lush meadow with autumn trees. Mid-match, lava creeps inward from the edges burning trees close by.

| Start | Mid-match |
|---|---|
| <img width="2030" height="1524" alt="greenreach" src="https://github.com/user-attachments/assets/e7e2e660-fb32-4ea4-a909-85b4bea9ee84" /> | <img width="2030" height="1517" alt="greenreach_evo" src="https://github.com/user-attachments/assets/99bb2469-f0c8-409e-93ad-735105702492" /> |

---

### Coralpoint

A sandy beach with water patches. Over time those patches freeze into ice and freezing close by trees.

| Start | Mid-match |
|---|---|
| <img width="2030" height="1527" alt="coralpoint" src="https://github.com/user-attachments/assets/99d14ab8-d183-4fda-812a-a268f09d345a" /> | <img width="2035" height="1525" alt="coralpoint_evo" src="https://github.com/user-attachments/assets/05e8ac9c-6df2-4ec0-aca1-577c8c70cb70" /> |

---

### Tundra

A snow and ice biome. Lava vents randomly erupt across the floor during the match, making it impossible to claim safe ground.

| Start | Mid-match |
|---|---|
| <img width="2030" height="1528" alt="tundra" src="https://github.com/user-attachments/assets/8a3bb045-0c5b-4920-80df-26106e51f419" /> | <img width="2468" height="1831" alt="tundra_evo" src="https://github.com/user-attachments/assets/c56f4768-a9e4-4250-9426-252e6d60a323" />|

![Lava damage demo](assets/gifs/lava_damage.gif)

---

### Ironridge

A rocky stone arena. Flooding begins mid-match: water rises ring by ring from the outside in, shrinking the solid playable area and slowing movement near the borders.

| Start | Mid-match |
|---|---|
| <img width="2030" height="1522" alt="ironridge" src="https://github.com/user-attachments/assets/0d9efd2c-48f7-40f1-89db-0671ae372d40" /> | <img width="2030" height="1534" alt="ironridge_evo" src="https://github.com/user-attachments/assets/e2d9a1c0-8cdd-4d2f-8bbd-9aeef5eda2ad" /> |

![Water slowdown demo](assets/gifs/water_slow.gif)

---

## The Storm

On top of each map's own evolution, a **universal storm mechanic** closes in on every match. Starting at 30 seconds, a ring of poison cloud tiles appears at the border of the map. Every 30 seconds after that, another ring closes inward. Each ring sets both `TILE_FLAG_DAMAGE` and `TILE_FLAG_SLOW` on those tiles, draining HP and slowing movement.

The storm is map-agnostic and runs in parallel with the map's own evolution. In the late game, you are dealing with both the map's transformation and an ever-shrinking safe zone. It guarantees the match ends and rewards players who stay central and aggressive.

<table>
  <tr>
    <td valign="top"><img src="https://github.com/user-attachments/assets/1fc6ae7c-8fd2-4adb-bd88-0c99740c4248" alt="storm1" width="250" /></td>
    <td valign="top"><img src="https://github.com/user-attachments/assets/80808f90-1ab8-4d3d-b9da-e3a954284367" alt="storm2" width="250" /></td>
    <td valign="top"><img src="https://github.com/user-attachments/assets/afa4a61d-580c-4834-b271-a51837d45a79" alt="storm3" width="250" /></td>
    <td valign="top"><img src="https://github.com/user-attachments/assets/3ca5afe0-8509-4e92-9050-15385c46d654" alt="storm4" width="250" /></td>
    <td valign="top"><img src="https://github.com/user-attachments/assets/068122eb-d3da-44f0-be7e-d704b9f598de" alt="storm5" width="250" /></td>
  </tr>
</table>


---

## Combat System

The combat system rewards timing and positioning over mashing.

**Melee** uses a weapon hitbox calculated each frame from the player's facing direction. If it overlaps an enemy's hitbox during an attack animation, damage applies. Light attack deals 10 damage, heavy deals 18.

**Blocking** absorbs a melee hit entirely and plays a sword-clash sound. It has a cooldown so it cannot be held indefinitely.

**Dashing** gives a brief speed burst with a 60-frame cooldown, useful for closing distance or escaping a corner.

**Ranged attacks** fire an arrow that checks collision against all players every frame. The AI uses this too, so tracking incoming arrows matters.

**Health potions** spawn every 15 seconds, up to 2 active at once. Walking over one restores health. They are a meaningful strategic target in both PVP and PVE, and the AI actively navigates toward them when its health is low.

---

## Player Names and HUD

Before the match, both players type a custom name using the keyboard. The screen handles backspace, caps lock, and length limits. Names default to "P1" and "P2" if skipped.

In-game, each name renders directly above its player's health bar. The position is calculated from the string length so it always sits centered over the bar regardless of how long the name is.

<p align="center">
  <img width="600" height="451" alt="name_enter" src="https://github.com/user-attachments/assets/495e32ae-c7a2-4572-affa-a6e665639b99" />
</p>

---

## Audio System and Interrupts

Getting audio to sound clean on bare-metal RISC-V is harder than it looks. Calling an audio update function from the main game loop creates audible pops and gaps because loop timing is not consistent enough to keep the hardware FIFO fed.

The solution was a dedicated **hardware interrupt** for audio. A second hardware timer is configured to fire at 8 kHz. Every time it overflows, it raises an interrupt on IRQ17. The processor jumps to the exception handler and the audio ISR runs immediately, regardless of what the game loop is doing at that moment.

Inside the ISR, samples are pushed to the audio FIFO from a mixer that combines background music and up to 8 simultaneous sound effect channels. An ISR budget cap keeps its runtime bounded so it cannot starve the game loop even if the FIFO has fallen behind.

The result is audio that is completely decoupled from game loop timing. A slow frame has no effect on audio quality.

All audio assets (sword swing variants, arrow sounds, block clashes, item pickups, background music, ambient storm audio) are stored as raw PCM arrays compiled directly into the binary.

---

## Decoration System

Maps would look sparse with just tiles. The decoration system fills them out with a budget-based procedural placement algorithm that runs once at map initialization.

Each map has a config struct that describes its visual character: how many trees and rocks to place, whether to cluster them or spread them out, whether to prefer large or mixed sizes, and which visual variants to use (autumn trees, ice trees, grey rocks, etc.). The algorithm reads this config, randomly drops clusters of decorations while respecting the budget, and avoids spawning on solid tiles or player spawn points.

**Canopy rendering** is one of the more satisfying details. Tall decorations like large trees and big rocks are split into two draw passes. The base is drawn first, behind players. The canopy top is drawn in a second pass, on top of players. This means you can actually hide behind a tree or duck behind a boulder during a fight. The canopy pass only runs for decorations near a player, so it does not cost much.

Adding a new decoration type is one entry in a lookup table. The placement algorithm and canopy system pick it up automatically.

Current decoration types: green trees (large/medium/small), autumn red and yellow trees, stick trees, ice trees, brown and grey rocks, bushes, ferns, cattails.

---

## Expandability

One of the design goals from the start was making it straightforward to add content without touching core systems.

**Adding a new map** takes four things: a tile layout grid, a config entry describing its visual style, an evolution type assignment, and its name on the map select screen. The renderer, obstacle system, decoration system, and storm all pick it up automatically from there.

**Adding a new evolution type** means writing an update function that calls `map_change_tile(row, col, sprite, flags)` and registering it. The tile flag system handles all gameplay effects.

**Adding a new decoration** is one table entry. One place to change.

**Adding a new entity type** means defining update and draw functions and assigning a type enum value. The flat entity pool handles lifecycle.

This structure mattered in practice. The ice map was added mid-project and required almost no changes outside its own files.

---

## Reinforcement Learning AI

The AI opponent uses a small neural network trained with reinforcement learning via **Gymnasium** in Python, then exported as C float arrays and run entirely in C on the RISC-V processor. No external library, no runtime Python, no dynamic allocation. Just matrix multiplication and ReLU, running deterministically within a bounded time window every frame.

### Training Approach

We built a Python simulation environment that mirrored the game's physics and combat rules. The agent was trained using PPO (Proximal Policy Optimization) from Stable-Baselines3.

Training was staged intentionally:

**Stage 1: Movement only.** The reward was simply for reducing distance to the opponent. No attack signals, nothing complex. The goal was to establish stable movement behavior before adding combat. This foundation made later training significantly more stable than trying to learn everything simultaneously from scratch.

**Stage 2: Combat rewards.** Once movement was solid, we added reward shaping for landing hits, taking damage, using block against incoming attacks, and survival. The agent learned to use its full action space.

### Observation Space (17 inputs)

Each frame the agent receives its own position, the opponent's position, both health values, all cooldown states, facing direction, and environmental information: the location and availability of the nearest health potion, and the distance of the nearest incoming arrow.

### Action Space

10 discrete actions: move up, down, left, right, light attack, heavy attack, ranged attack, dash, block, idle.

### Network Architecture

Three fully connected layers: 17 inputs, two hidden layers of 64 neurons with ReLU activations, 10 outputs with argmax selection.

After training, weights are exported as C float arrays and compiled into the binary. The observation vector in C matches the Python training environment exactly, including all normalization ranges. Keeping those aligned was one of the more tedious parts of the port.

---

## Performance Optimizations

Running a full game loop at 60fps on bare-metal RISC-V without hardware acceleration requires being deliberate about where time goes.

**Double-buffered rendering** keeps the display tearing-free. Two VGA buffers are maintained, one displayed while the other is drawn to. `wait_for_vsync()` swaps them atomically.

**Dirty-rect entity erasure** avoids clearing the full screen each frame. Only the pixels covered by moving entities are erased. Each entity stores its position from the previous two frames (one per buffer) so the correct pixels are cleaned regardless of which buffer is active.

**Interrupt-driven audio** decouples sound from game loop timing entirely. A slow frame has no effect on audio quality.

**ISR budget cap** bounds the audio ISR's runtime so it cannot steal too much time from the game loop even when catching up.

**Canopy culling** only runs the second-pass canopy draw for decorations near a player, avoiding unnecessary sprite blits.

**Transparency via color key** uses magenta (RGB565 `0xF81F`) as a transparent pixel value. The renderer skips those writes rather than overdrawing background tiles.

---

## Hardware Overview

| Component | Details |
|---|---|
| Processor | Nios V/RISC-V soft-core on Cyclone V FPGA |
| Display | VGA, 320x240, RGB565, pixel buffer at `0x08000000` |
| Input | PS/2 keyboard, polled via FIFO each frame |
| Audio | 8 kHz PCM, hardware ISR on IRQ17 |
| Frame pacing | Hardware timer on IRQ16, sets `frame_flag` each frame |
| Double buffer | Two 512x240 pixel buffers, swapped via `PIXEL_BUF_CTRL` |

All peripheral access is through memory-mapped I/O using the Cyclone V address map.
