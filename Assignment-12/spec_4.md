# 🦀 Clawdius the Crab — Spike Jumper Game Design Specification

---

## 1. Overview

**Clawdius the Crab** is an endless side-scrolling runner/jumper in the tradition of the Chrome Dinosaur game. The player controls a pastel pink crab that scuttles left-to-right through a series of biomes — the ocean floor, the deck of a ship, a beach, and inside a seafood restaurant — avoiding obstacles and enemies unique to each environment. The game features day/night cycles, escalating difficulty, optional rhythm-runner mode, async ghost multiplayer, and a level editor.

---

## 2. Visual Design

### 2.1 Color Palette

| Element | Color |
|---|---|
| Player crab (Clawdius) | Pastel pink `#F4A7B9` |
| Ocean background (day) | Deep teal `#1A6B8A` → aqua `#5BC8D4` gradient |
| Ocean background (night) | Navy `#0B1F3A` → dark teal `#1A3A4A` gradient |
| Ship deck (day) | Warm oak `#C89A5A`, grey metal accents |
| Ship deck (night) | Shadowed oak `#6B4F2A` with lantern glow |
| Beach (day) | Sandy gold `#F5E0A0`, sky `#87CEEB` |
| Beach (night) | Moonlit sand `#C8C090`, dark sky `#1A1A3A` |
| Seafood restaurant | Warm red-checkered floor, dim amber lighting |
| Evil crabs | Vivid red `#E03030` |
| UI / HUD | Semi-transparent dark panels, white text |

### 2.2 Parallax Layers (per biome)

**Ocean Floor**
- Layer 1 (far background): Drifting jellyfish silhouettes, distant coral formations, light caustic shimmer
- Layer 2 (mid background): Mid-depth coral reef, kelp swaying
- Layer 3 (near background): Sand dunes and pebbles slowly scrolling
- Layer 4 (foreground): Fine sand particles and occasional bubbles rising

**Ship Deck**
- Layer 1: Open sky / ocean horizon
- Layer 2: Distant shoreline or open sea
- Layer 3: Ship railing and rigging ropes
- Layer 4: Ship deck planks with rope coils and barrels as decor

**Beach**
- Layer 1: Sky and horizon
- Layer 2: Ocean waves gently rolling in
- Layer 3: Distant palm trees
- Layer 4: Sand surface with shells and footprints

**Seafood Restaurant**
- Layer 1: Restaurant wall with framed fish art / menu boards
- Layer 2: Tables and chairs in the background
- Layer 3: Tablecloth-covered tables, candles flickering
- Layer 4: Tiled floor with the player running across it

### 2.3 Day/Night Cycle

- Full cycle takes approximately 4–5 minutes of gameplay
- Gradual sky/water color lerp over ~30 seconds at transitions
- Night adds ambient glow effects: bioluminescent particles in ocean, lantern halos on ship, restaurant lighting stays warm/constant
- Enemy sprites gain slight glow outlines at night for visibility
- **No sun or moon drawn.** Brightness is handled entirely via the sky gradient and a subtle day/night brightness overlay (bright tint during day, dark overlay at night).
- Stars appear at night (no moon silhouette)

### 2.4 Enhanced Parallax Backgrounds

All parallax layer speeds have been reduced to minimize excessive movement. Deterministic per-element values are used for animated details to prevent per-frame flickering.

**Ocean Floor (enhanced)**
- Layer 0: Jellyfish silhouettes with gentle reduced bob (amplitude ×3 vs prior ×12), plus drifting fish silhouettes in background
- Layer 1: More coral variety (larger count, randomized heights), small coral clusters, kelp strands with subtle wave animation
- Layer 2: Sand dunes, scattered pebbles, rising bioluminescent bubble particles
- Layer 3: Fine foreground sand

**Ship Deck (enhanced)**
- Layer 0: Multiple cloud formations in sky
- Layer 1: Rigging ropes, distant seagull silhouettes in sky
- Layer 2: Ship railing and deck planks
- Layer 3: Barrels and rope coils

**Beach (enhanced)**
- Layer 0: Multiple cloud formations (layered with varying opacity), wave lines
- Layer 1: Distant fish silhouettes in the water, ocean wave lines
- Layer 2: Palm trees, scattered shells on sand
- Layer 3: Sand surface

**Seafood Restaurant (enhanced)**
- Layer 0: Framed fish artwork on walls
- Layer 2: Tables with flickering candle glow
- Layer 3: Animated checkered floor

---

## 3. Player Mechanics

### 3.1 Movement

- Clawdius runs automatically from left to right at a base speed that scales up over time
- Player input: **tap / spacebar / up arrow** to jump
- **Double-jump:** Press/tap a second time while airborne for a second, slightly lower jump. Indicated by gold sparkles on Clawdius's claws. Second jump velocity: ~460 (first jump: ~530).
- Clawdius scuttles using a 6-frame leg cycle animation that speeds up with game speed

### 3.2 Jump Physics

- Jump uses a parabolic arc with:
  - **Rise:** Fast acceleration upward, easing out (ease-out cubic)
  - **Peak:** Brief float pause (~3 frames)
  - **Fall:** Gravity pulls down with easing in (ease-in cubic), faster than rise to feel snappy
- Jump height: ~2.5× Clawdius's body height
- Horizontal position locked; only vertical movement on jump
- Coyote time: 80ms window to jump after walking off a ledge
- Jump buffer: Input registered up to 100ms before landing triggers a queued jump

### 3.3 Animation States

| State | Description |
|---|---|
| `idle` | Subtle body bob, antenna twitch every 2s |
| `run` | 6-frame leg scuttle cycle, synchronized to speed |
| `jump_squat` | 2-frame squash before launch (body widens, lowers) |
| `jump_ascent` | Stretch vertically, claws tuck up |
| `jump_peak` | Neutral, slight spread of claws |
| `jump_descent` | Tuck claws downward, slight forward lean |
| `land` | Quick squash on landing, spring back to run |
| `death` | Biome-specific death animation (see §6) |
| `transition` | Walking animation while biome transition scroll plays |

### 3.4 Squash & Stretch

- **Pre-jump squat:** 100ms before launch: scaleX × 1.3, scaleY × 0.7
- **Launch:** scaleX × 0.75, scaleY × 1.35
- **Landing:** scaleX × 1.25, scaleY × 0.8 → spring to 1,1 over 120ms
- All transforms originate from the bottom-center pivot (feet stay on ground)

---

## 4. Biomes & Transitions

### 4.1 Biome Order

Biomes cycle in a repeating pseudo-random rotation:

`Ocean → Ship → Ocean → Beach → Ocean → Restaurant → Ocean → ...`

The ocean sections are always between other biomes and grow shorter as difficulty increases (less breathing room).

### 4.2 Transition Sequences

**Ocean → Ship:** A wooden ship hull scrolls in from the right. Clawdius scuttles up an anchor chain and onto the deck. (~3 seconds of animated climb, player cannot die.)

**Ship → Ocean:** Clawdius reaches the stern, scuttles down a rope, and splashes back into the water.

**Ocean → Beach:** A sandy shore slides in. Clawdius walks out of the surf onto the sand.

**Beach → Restaurant:** An open restaurant door appears. Clawdius scuttles inside.

**Restaurant → Ocean:** Clawdius exits through a back kitchen door that leads to a dock, then drops into the ocean.

---

## 5. Hazard Types & Behaviors

### 5.1 Ocean Floor Hazards

| Hazard | Type | Behavior |
|---|---|---|
| **Sea Urchin** | Ground | Static. Sits on the seafloor. Single or clusters of 2–3. Spikes extend slightly in a pulse animation. |
| **Lunge Fish** | Air | Spawns off-screen above or at the right edge. Dives diagonally downward toward Clawdius's predicted position. Gives a 0.4s "windup" telegraph (fish wiggles before lunging). |
| **Anemone** | Ground | Slow-moving, expands and contracts. Hitbox only active when expanded. |

### 5.2 Ship Deck Hazards

| Hazard | Type | Behavior |
|---|---|---|
| **Seagull** | Air | Swoops in from upper right in an arc toward Clawdius. Gives a screech audio cue and shadow on ground as telegraph. |
| **Thrown Knife** | Ground/Low-air | Slides across the deck floor or tumbles at low altitude. Some knives bounce once. |
| **Coiled Rope** | Ground | Static obstacle, low profile — requires a jump. |

### 5.3 Beach Hazards

| Hazard | Type | Behavior |
|---|---|---|
| **Evil Crab (Red)** | Ground | Scuttles toward Clawdius at variable speed. Can also move laterally to cut off the jump landing zone. |
| **Kid's Arm** | Air/Grab | A child's arm (long skin-colored rectangle extending off the top of the screen) descends with the hand attached, trying to scoop Clawdius. The arm visually connects to something off-screen above, so it no longer looks like a floating hand. Gives a shadow on ground as warning. Lingers briefly before retracting if missed. |

### 5.4 Restaurant Hazards

| Hazard | Type | Behavior |
|---|---|---|
| **Crab Mallet** | Ground | Sits on the floor. May spin slowly. Single or pairs. |
| **Crab Pick / Fork** | Ground | Slides across the floor from the right at varying speeds. |
| **Giant Butcher's Blade** | Air | A massive cleaver descends from the top of the screen on a long wooden handle (which extends off-screen above). The blade is a large silver trapezoid with a sharp cutting edge at the bottom, bolster, and rivets. Gives a shadow on ground as telegraph. If it hits Clawdius, he's chopped. |

### 5.5 Hazard Hitboxes

All hitboxes are 15–20% smaller than the visible sprite to allow "close call" moments. Grab hazards (hands, arm) have a slightly larger hitbox than damage hazards to feel satisfying to dodge.

---

## 6. Death Sequences & Game Over Screens

### 6.1 Ocean Death — Octopus

**Death trigger:** Hit by sea urchin or lunge fish. Particle burst (pink shell fragments + bubbles). Screen shake (0.3s). Clawdius goes limp.

**Death animation:** An octopus shoots in from the left, wraps a tentacle around Clawdius, and drags him off the right edge.

**Game Over Screen:** An octopus cave entrance in rocky seafloor. Crab exoskeleton (empty shell, cracked) sits outside the cave. Octopus tentacles lurk partially visible in the cave shadow. Dim bioluminescent ambient glow.

### 6.2 Ship Death — Seagull

**Death trigger:** Hit by seagull or knife. Particle burst (pink shell + feathers).

**Death animation:** A large seagull swoops down, snatches Clawdius in its beak, and flies off screen.

**Game Over Screen:** Stylized sunset horizon. Silhouette of a seagull flying away with Clawdius dangling in its beak.

### 6.3 Beach Death — Evil Crab

**Death trigger:** Touched by evil crab. Particle burst (sand + shell fragments).

**Death animation:** Clawdius flips upside down and is dragged off screen left by two red crabs.

**Game Over Screen:** Two red crab claws crossed in an X shape on sandy background.

### 6.4 Beach Death — Kid's Hand Grab

**Death trigger:** Grabbed by kid's hand. Clawdius is lifted upward off screen.

**Game Over Screen:** A cheerful little kid walking hand-in-hand with their mom along a beach. The kid holds a colorful plastic sand bucket with Clawdius visible inside it, one claw poking over the rim.

### 6.5 Restaurant Death — Tool or Arm Grab

**Death trigger:** Any restaurant hazard. Particle burst (shell fragments + steam wisps).

**Death animation (tools):** Clawdius is knocked and tumbles off screen.
**Death animation (blade):** The giant cleaver slams down onto Clawdius, sending him off screen.

**Game Over Screen:** A beautifully plated crab dish — drawn in the game's art style — on a white restaurant plate with garnish. Elegant and darkly comedic.

### 6.6 Game Over UI (All Biomes)

- **Death screen elements:**
  - Dark semi-transparent overlay (no biome artwork background — removed for cleanliness)
  - Large "YOU DIED" text — color-coded per biome: ocean=blue, ship=orange, beach=yellow, restaurant=red-orange
  - Biome-specific flavor text ("Dragged into the deep...", "On the menu tonight...", etc.)
  - Distance traveled: `🦀 You traveled: [X] meters`
  - High score display: `🏆 Best: [Y] meters`
  - Button: **▶ Try Again** (restarts from beginning)
  - Button: **🏠 Main Menu**
  - Particle effects continue playing softly in the background

---

## 7. Pattern Library (Obstacle Chunks)

Each "chunk" is a pre-authored section of obstacle patterns that the generator selects from based on current difficulty tier.

| # | Name | Biome | Description |
|---|---|---|---|
| 1 | **Urchin Gap** | Ocean | Two urchins with a jumpable gap between them. Teaches basic timing. |
| 2 | **Fish Ambush** | Ocean | One lunge fish swoops in 0.5s after a ground urchin, requiring a jump that must land and quickly recover or jump again. |
| 3 | **Urchin Wall** | Ocean | Three urchins in a tight cluster requiring a single long jump to clear all three. |
| 4 | **Double Lunge** | Ocean | Two fish lunge in rapid succession from different angles, testing jump timing. |
| 5 | **Gull Dive + Knife** | Ship | A seagull swoops while a knife slides on the deck — player must time a jump that avoids both. |
| 6 | **Knife Bounce Pair** | Ship | Two bouncing knives with different bounce heights require a jump timed to the arc gap. |
| 7 | **Evil Crab Chase** | Beach | Two red crabs of different speeds approach, requiring a jump that clears the faster one. |
| 8 | **Hand + Crab Pincer** | Beach | Hand descends from above while a red crab approaches from the right — jump must be timed to avoid the hand on the way up AND land clear of the crab. |
| 9 | **Mallet Gauntlet** | Restaurant | Four mallets spaced irregularly on the floor in rapid succession. Tests rhythm. |
| 10 | **Arm + Fork Combo** | Restaurant | A fork slides fast while an arm descends — short hop clears the fork, but jumping too high risks the arm. |
| 11 | **Anemone Timing** | Ocean | Two anemones pulsing out of sync — player must wait for the gap window. |
| 12 | **Rope + Gull** | Ship | Low rope requires a small hop; a gull arrives immediately after, punishing high jumps. |

At higher difficulty tiers, chunks are combined into longer sequences and spacing is reduced.

---

## 8. Juice Effects

### 8.1 Squash & Stretch
*(See §3.4 for full detail)*
- Pre-jump squat, launch stretch, landing squash with spring recovery

### 8.2 Particles

| Trigger | Particles |
|---|---|
| Jump | 4–6 small sand/bubble puffs from feet |
| Landing | 8–12 dust/sand/water droplets fan outward |
| Death (all) | 15–20 shell fragment shards, biome-specific color accent particles |
| "Perfect" rating (rhythm mode) | Gold star burst with shimmer trail |
| "Good" rating (rhythm mode) | White ring pulse |
| "Miss" rating (rhythm mode) | Small grey X pop |
| Biome transition entry | Bubble burst (ocean), confetti (beach), spark (restaurant) |

### 8.3 Screen Shake

| Trigger | Intensity | Duration |
|---|---|---|
| Death | Medium | 0.35s |
| Heavy landing (long fall) | Light | 0.15s |
| Lunge fish impact (near miss) | Micro | 0.08s |
| Seagull screech (audio cue) | Micro | 0.05s |

All screen shakes use exponential decay (strongest at onset, tapering to zero).

### 8.4 Visual Feedback

- **Near miss:** Bright flash around Clawdius (50ms white outline pulse) when a hazard passes within 10px
- **Speed increase notification:** Subtle vignette pulse on screen edges when speed tier increases
- **Night transition:** Gradual sky darkening with a rising moon animation

### 8.5 Audio Juice

| Sound | Trigger | Implementation |
|---|---|---|
| 🔊 **Boing** | Jump launch (pitch ±5% per jump, +30% for double jump) | Web Audio API synthesized: sine oscillator, 300→95Hz exponential ramp, 0.3s |
| 🔊 **Boing (higher)** | Double-jump launch | Same synth at 1.3× pitch |
| 💥 **Shell crack** | Death | Web Audio API noise burst with exponential envelope, 0.45s |
| 🎵 **Menu music** | Main menu, normal play, rhythm mode | `ScreenRecording_02-26-2026 22-41-05_1.mov` looped audio, toggleable |
| 🐦 **Seagull screech** | Seagull spawn telegraph | (planned) |
| 🐟 **Fish whoosh** | Lunge fish dive | (planned) |
| 👏 **Perfect chime** | Rhythm mode perfect hit | Gold particle burst (visual) |

### 8.6 UI Animations

- Buttons scale to 1.05× on hover, 0.95× on press, with 80ms ease
- Score counter uses rolling digit animation
- Distance counter increments with a subtle font-weight pulse

---

## 9. Difficulty Progression

### 9.1 Speed Curve

| Time Elapsed | Speed Multiplier | Feel |
|---|---|---|
| 0–30s | 1.0× | Tutorial / warm-up |
| 30s–1:30 | 1.0× → 1.4× | Moderate, manageable |
| 1:30–3:00 | 1.4× → 1.9× | Noticeably faster |
| 3:00–5:00 | 1.9× → 2.5× | Challenging |
| 5:00+ | 2.5× → 3.5× cap | Expert |

Speed increases are smooth and continuous, not step-based, using a logarithmic curve.

### 9.2 Obstacle Density

- Chunks spawn with increasing frequency over time
- Early game: single obstacles, long gaps
- Mid game: paired obstacles, shorter gaps, introduced combo chunks
- Late game: complex multi-hazard chunks from the full pattern library

### 9.3 Pattern Selection

A weighted random system pulls chunks from the pattern library. Earlier chunks (simpler) have high weight early and fade; complex chunks gain weight over time. The system prevents repeating the same chunk more than twice in a row.

---

## 10. User Interface

### 10.1 Main Menu

- **Background:** Animated ocean loop with Clawdius scuttling across the title screen, decorative coral row along ground
- **Title:** "CLAWDIUS" in bubbly stylized font with subtle wave distortion
- **Button layout (3-row grid):**
  - Row 1 (full width): ▶ **PLAY** — starts standard endless run
  - Row 2: 🎵 **Rhythm Runner** | 👻 **Ghost Race**
  - Row 3: 🗺️ **Level Editor** | 🔇 **Music: OFF/ON** (toggleable — always initializes to OFF every page load, so button always opens showing **Music: OFF**)
- **Music:** Plays `ScreenRecording_02-26-2026 22-41-05_1.mov` as background audio (looped, volume 0.6). `musicOn` initializes to `false` on every page load — no localStorage read — so the button label always truthfully matches the actual audio state. User can toggle ON during a session.
- **Hint text:** Explains Space/Click/Tap to jump and double-tap for double jump

### 10.2 HUD (In-Game)

- Top-left: Distance counter `🦀 [X]m`
- Top-right: Best distance `🏆 [Y]m`
- Bottom-center (rhythm mode only): Beat accuracy bar + score/combo panel
- Subtle biome name tooltip in top-center

### 10.2 HUD (In-Game)

- Top-left: Distance counter `🦀 [X]m`
- Top-right: Best distance `🏆 [Y]m`
- Bottom-right (rhythm mode only): Beat accuracy indicator
- Subtle biome name tooltip fades in on biome transition
- Day/night icon (sun/moon) in top-center, fades out after 2s

### 10.3 Pause Screen

- Tap Escape / back button to pause
- Shows: Resume, Restart, Main Menu, Music Toggle

### 10.4 Game Over Screen
*(See §6.6)*

---

## 11. Audio

### 11.1 Music

Each biome has a unique looping music track:

| Biome | Music Style |
|---|---|
| Ocean (day) | Upbeat, bubbly synth pop with marimba |
| Ocean (night) | Slower, ethereal bioluminescent ambient |
| Ship (day) | Jaunty sea shanty / accordion |
| Ship (night) | Mysterious nautical minor-key |
| Beach (day) | Tropical, steel drum, upbeat |
| Beach (night) | Calmer, moon-lit lo-fi |
| Restaurant | French café jazz / bossa nova |

Music crossfades over 2 seconds between biomes. Music toggle is **session-only** (not persisted via localStorage). `musicOn` always starts `false` on page load. **Defaults to OFF** — user must opt in every session by clicking the Music button.

### 11.2 Sound Effects
*(Full list in §8.5)*

All SFX use Web Audio API (or equivalent) for low latency. Jump boing has a slight pitch randomization to prevent repetition fatigue.

---

## 12. High Score Persistence

- High score (best distance in meters) stored in `localStorage` with key `clawdius_highscore`
- Displayed on game over screen and main menu
- Ghost run data stored as compressed JSON array of `[timestamp, y_position]` pairs in `localStorage` with key `clawdius_ghost_[run_id]`
- Up to 5 ghost runs stored; oldest overwritten when limit reached
- Rhythm mode has a separate high score key: `clawdius_rhythm_highscore`

---

## 13. Rhythm Runner Mode (Optional)

### 13.1 Overview

An optional game mode toggled from the main menu. Obstacles are spawned at precise timestamps aligned to the beat of the background music track. Scoring rewards rhythmic accuracy.

### 13.2 Beat Detection & Music Analysis

- Pre-authored beat maps are bundled with each biome music track (JSON files defining beat timestamps in milliseconds)
- Optional: real-time beat detection fallback using energy-based onset detection on the audio stream (FFT peaks in bass frequency band 60–180Hz)
- Beat map editor built into the Level Editor (see §15)

### 13.3 Obstacle Timing

- Each beat has a corresponding obstacle spawn with type defined by the beat map
- Obstacles reach Clawdius's position at exactly the beat hit moment
- A visual "approach bar" UI element shows upcoming beats as circles traveling toward a hit line

### 13.4 Scoring

| Rating | Timing Window | Points | Visual Feedback |
|---|---|---|---|
| **PERFECT** | ±30ms of beat | 300 pts | Gold star burst |
| **GOOD** | ±80ms of beat | 100 pts | White ring pulse |
| **MISS** | Outside window | 0 pts | Grey X pop |

- Multiplier: 1× base, increases by 0.5× per consecutive PERFECT/GOOD, resets on MISS
- End screen shows: accuracy percentage, total score, best score, grade (S/A/B/C/F)

---

## 14. Multiplayer Ghost Racing

### 14.1 Overview

Players can race asynchronously against recorded ghost runs from friends. No real-time networking required.

### 14.2 Ghost Data Format

Each ghost run stores:
```json
{
  "run_id": "uuid",
  "player_name": "string",
  "date": "ISO timestamp",
  "distance": 1234,
  "frames": [[time_ms, y_position, biome_id], ...]
}
```

Frames are sampled at 60fps and compressed with run-length encoding for storage efficiency.

### 14.3 Sharing

- Ghost runs can be exported as a base64-encoded string (copy to clipboard)
- Friends paste the string into "Import Ghost" on the Ghost Race screen
- Up to 3 ghost crabs displayed simultaneously, each a different tint (blue, yellow, green)
- Ghost crabs are semi-transparent and cannot affect gameplay

### 14.4 Ghost Race UI

- Main menu → Ghost Race → shows saved ghosts with name, date, distance
- "Import Ghost" button to paste a shared ghost string
- "Race!" launches the game with selected ghost(s) active
- HUD shows ghost position(s) relative to Clawdius

---

## 15. Level Editor

### 15.1 Overview

A full-featured level editor accessible from the main menu. Players can create custom runs with hand-placed obstacles, custom biome sequences, and authored beat maps.

### 15.2 Features

- Timeline view: scrub through level time axis, place obstacle chunks or individual hazards at timestamps (0–20 s)
- Biome selector: choose which biome background is active during the preview run
- Obstacle palette: **all 11 hazard types** from §5 are available regardless of biome — any enemy can be placed in any biome (e.g. a Lunge Fish in the Restaurant, or a Giant Butcher's Blade in the Ocean)
- Beat map editor: import audio, tap to mark beats, auto-detect beats, manually adjust
- Preview: clicking **▶ Preview** launches the selected biome with obstacles spawning at exactly the authored timestamps. Biome cycling and random enemy spawning are both suppressed — only the authored pattern plays back. Enemies are created at the right edge of the screen when `elapsed` time reaches their placed timestamp.
- **Cross-biome enemy freedom:** The editor imposes no biome restriction on enemy choice. Fish can swim through a restaurant; mallets can appear on the ocean floor. All 11 types draw and behave identically to how they do in the main game.
- Save/Export: levels saved to `localStorage` and exportable as JSON strings
- Share: copy a share code that friends can import on the Ghost Race / Level Editor screen

### 15.3 Level Library

- "Community" tab shows featured imported levels
- Each level displays: creator name, length, difficulty rating, play count

---

## 16. Technical Notes

- Target: browser-based (HTML5 Canvas or WebGL via Phaser/PixiJS, or vanilla Canvas)
- Target frame rate: 60fps
- Designed for both desktop (keyboard) and mobile (tap)
- Audio: Web Audio API for SFX timing precision
- Storage: `localStorage` for scores, ghosts, levels, settings
- Art style: 2D pixel art or vector illustration (hand-drawn feel), consistent ~48px tile grid

---

*End of Specification — Clawdius the Crab v1.0*
