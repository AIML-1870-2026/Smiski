# 🪼 Moon Jelly Color Mixer — Product Specification

**Version:** 1.1
**Aesthetic:** Frutiger Aero / Y2K Nostalgia  
**Concept:** An interactive color theory tool featuring translucent moon jellyfish as living color swatches, blending together like real light or ink.

---

## Table of Contents

1. [Overview](#overview)
2. [Visual Design Language](#visual-design-language)
3. [App Structure & Navigation](#app-structure--navigation)
4. [Tab 1 — Moon Jelly Color Mixer](#tab-1--moon-jelly-color-mixer)
   - [RGB Mode](#rgb-mode)
   - [CMYK Mode](#cmyk-mode)
   - [Mixer Controls](#mixer-controls)
   - [Result Display](#result-display)
5. [Tab 2 — Palette Studio](#tab-2--palette-studio)
   - [Saved Palettes](#saved-palettes)
   - [Color Harmony Generator](#color-harmony-generator)
   - [Random & Smart Color Generator](#random--smart-color-generator)
6. [Animations & Interactions](#animations--interactions)
7. [Data Model](#data-model)
8. [Technical Stack](#technical-stack)
9. [Accessibility](#accessibility)
10. [Stretch Challenges](#stretch-challenges)
    - [Contrast Checker](#contrast-checker)
    - [Color Blindness Simulator](#color-blindness-simulator)
    - [Accessible Palette Mode](#accessible-palette-mode)

---

## Overview

Moon Jelly Color Mixer is a single-page web application with two main tabs. Users mix colors by dragging translucent jellyfish together, controlling overlap via sliders, and saving the results to named palettes. The app supports both **RGB** (additive, light-based) and **CMYK** (subtractive, ink-based) color models, each with its own environmental theme.

---

## Visual Design Language

### Aesthetic: Frutiger Aero
- **Glassy, translucent UI elements** — frosted glass panels, soft glows, subtle reflections
- **Gradients everywhere** — aqua-to-white, deep blue-to-teal, cool sky blues
- **Rounded corners** on all containers (border-radius: 16–32px)
- **Soft drop shadows** and inner glows
- **Bubbly typography** — rounded sans-serif fonts (e.g., Nunito, Comfortaa, or Varela Round)
- **Lens flares and bloom effects** subtly present on UI highlights
- **Nature-inspired palette** — sky blues, seafoam, white, pearlescent highlights

### Icons & Decorative Elements
- Small jellyfish silhouettes as favicon and loading indicators
- Bubble particle effects on hover/click
- Animated water caustics (subtle, CSS or canvas-based) in backgrounds
- Pearl/glass-like buttons with specular highlights

---

## App Structure & Navigation

```
┌─────────────────────────────────────────────────────────┐
│  🪼  Moon Jelly Color Mixer                [RGB] [CMYK] │
│─────────────────────────────────────────────────────────│
│  [ 🌊 Color Mixer ]    [ 🎨 Palette Studio ]            │
│─────────────────────────────────────────────────────────│
│                                                         │
│                  [Active Tab Content]                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Header
- App name and logo (small jellyfish icon)
- **Mode toggle**: `RGB` / `CMYK` pill-style toggle, visible at all times
  - Switching mode changes the background environment and jellyfish count/colors
- Two main tab buttons: **Color Mixer** and **Palette Studio**

---

## Tab 1 — Moon Jelly Color Mixer

### RGB Mode

**Environment: Sunlit Ocean**
- Background: animated ocean scene in daylight — shafts of light filtering through blue-green water, soft caustic light patterns rippling across the scene
- Color palette: cerulean, aquamarine, white highlights, golden light rays
- Ambient particles: small bubbles floating upward

**Jellyfish:**
Three primary RGB jellyfish, each a fully translucent bell rendered in pure color with no white highlight or specular overlay — so the blended color in overlap zones is clearly visible:

| Jelly | Color | Glow Color |
|-------|-------|------------|
| Red Jelly | `rgb(255, 0, 0)` | soft coral/salmon glow |
| Green Jelly | `rgb(0, 255, 0)` | mint glow |
| Blue Jelly | `rgb(0, 0, 255)` | electric blue glow |

- Mix behavior follows **additive color model**:
  - Red + Green = Yellow
  - Red + Blue = Magenta
  - Green + Blue = Cyan
  - Red + Green + Blue = White

---

### CMYK Mode

**Environment: Bioluminescent Night Ocean**
- Background: deep dark ocean at night — near-black water, stars visible at the surface, bioluminescent particles, glowing coral and anemones in the distance
- Color palette: deep navy, near-black, vivid neon glows
- Jellyfish emit visible light halos; their glow is the dominant visual feature

**Jellyfish:**
Four primary CMYK jellyfish, also rendered without specular highlight ellipses so blended overlap colors read clearly:

| Jelly | Color | Glow Color |
|-------|-------|------------|
| Cyan Jelly | `rgb(0, 188, 212)` | electric teal glow |
| Magenta Jelly | `rgb(233, 30, 99)` | hot pink glow |
| Yellow Jelly | `rgb(255, 235, 59)` | golden glow |
| Key/Black Jelly | `rgb(30, 30, 30)` | dark violet aura |

- Mix behavior follows **subtractive color model**:
  - Cyan + Magenta = Blue
  - Cyan + Yellow = Green
  - Magenta + Yellow = Red
  - C + M + Y = near-Black (Key)

---

### Mixer Controls

#### Color Source — Drag OR Sliders (mutually exclusive)

The result color is driven by **one source at a time**:

| Source | When it activates | How it works |
|--------|------------------|--------------|
| **Jellyfish Overlap** | When two or more jellies are physically dragged to overlap on screen | The overlap region is detected geometrically; the blended color of every overlapping pair (and triple) is computed using the active color model and averaged by overlap area |
| **Sliders** | When the user adjusts a slider | Slider values directly set each channel's intensity and the pairwise / all-group mix weights; the result is calculated mathematically |

Switching from one input to the other immediately updates the result display. If no jellies overlap and no sliders have been changed, the result defaults to showing no mix (transparent / neutral).

#### Free Drag Interaction
- Jellyfish **float idly** in the scene with a gentle, organic bobbing animation (sinusoidal vertical drift + slight tentacle sway)
- Users can **click and drag** any jellyfish to reposition it
- When two or more jellyfish **overlap**, their translucent bodies blend visually; the **mixed color becomes the result** and is computed in real time from the overlap geometry
- The overlap region renders as a blended semi-transparent layer using the correct color model math — no white highlight is added so the true mixed hue is clearly visible
- Jellyfish return to idle floating animation when released
- Dragging activates **Drag mode**; any subsequent slider move switches to **Slider mode**

#### Sliders Panel
A Frutiger Aero frosted-glass card titled **"Mix Controls"** with a **Slider Mode** indicator. Moving any slider deactivates drag-driven color and recomputes via pure channel math.

**RGB Mode Sliders:**
- `Red Intensity` — 0 to 255
- `Green Intensity` — 0 to 255
- `Blue Intensity` — 0 to 255
- `Red–Green Overlap` — 0% to 100%
- `Red–Blue Overlap` — 0% to 100%
- `Green–Blue Overlap` — 0% to 100%
- `All Three Overlap` — 0% to 100%

**CMYK Mode Sliders:**
- `Cyan` — 0% to 100%
- `Magenta` — 0% to 100%
- `Yellow` — 0% to 100%
- `Key (Black)` — 0% to 100%
- Pairwise overlap sliders for each combination

Sliders are styled as glassy aqua pill-shaped tracks with pearl-like thumb handles.

---

### Result Display

Below the jellyfish scene (or in a side panel on wider screens), a **"Your Mix"** card displays:

```
┌─────────────────────────────────────────────────────┐
│  🪼  Your Mix                                        │
│                                                      │
│     [Animated Result Jellyfish]                      │
│         rendered in the mixed color                  │
│                                                      │
│     HEX: #A040FF                                     │
│     Name: "Amethyst Glow"                            │
│     RGB: 160, 64, 255                                │
│     CMYK: 37%, 75%, 0%, 0%                           │
│                                                      │
│  [ 💾 Save to Palette ]  [ 📋 Copy HEX ]             │
└─────────────────────────────────────────────────────┘
```

- **Result Jellyfish**: A single moon jellyfish rendered in the computed color, gently floating. In RGB mode it glows softly in daylight; in CMYK mode it bioluminesces in the dark.
- **Color Name**: Nearest named CSS / X11 color, or a poeticized name from a curated lookup table (e.g., "Deep Ocean Teal", "Coral Sunrise")
- **HEX code**: Displayed prominently, with one-click copy
- **RGB values** always shown
- **CMYK values** always shown (converted if needed)
- **Save to Palette** button: opens a small modal asking for a palette name (create new or add to existing)
- **Copy HEX** button: copies to clipboard with a small bubble animation confirmation

---

## Tab 2 — Palette Studio

### Saved Palettes

```
┌──────────────────────────────────────────────────────────┐
│  🎨 My Palettes                          [ + New Palette ]│
│──────────────────────────────────────────────────────────│
│                                                           │
│  ┌─ "Ocean Dreams" ────────────────────────────────────┐ │
│  │  🪼  🪼  🪼  🪼  🪼                                  │ │
│  │  #0D7C8F  #A8E6E0  #F7C6A3  #FF6B8A  #C9A0DC       │ │
│  │  [ Export ]  [ Rename ]  [ Delete ]                 │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                           │
│  ┌─ "Bioluminescent" ─────────────────────────────────┐  │
│  │  ...                                               │  │
│  └───────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

- Each palette is displayed as a row of small jellyfish swatches
- Clicking a swatch jellyfish opens its **Color Detail Panel** (see below)
- Palettes can be **named**, **renamed**, and **deleted**
- Export palette as: PNG swatch sheet, CSS variables, JSON

### Color Detail Panel

Clicking any saved color swatch opens a side panel or modal:

```
┌──────────────────────────────────────────────────────┐
│  🪼  Amethyst Glow            [ ← Back to Palette ]  │
│  #A040FF                                             │
│──────────────────────────────────────────────────────│
│  Generate Harmonies:                                 │
│  [ Complementary ]  [ Analogous ]  [ Triadic ]       │
│  [ Split-Complementary ]  [ Tetradic ]               │
│──────────────────────────────────────────────────────│
│  [Result displayed as row of jellyfish swatches]     │
│  Each with HEX + name, and individual Save buttons   │
└──────────────────────────────────────────────────────┘
```

**Harmony Types:**

| Harmony | Description | Colors Generated |
|---------|-------------|-----------------|
| Complementary | Opposite on the color wheel | 2 colors |
| Analogous | Adjacent on the wheel (±30°) | 3 colors |
| Triadic | Evenly spaced 120° apart | 3 colors |
| Split-Complementary | Base + two colors adjacent to its complement | 3 colors |
| Tetradic (Rectangle) | Two complementary pairs 90° apart | 4 colors |

- Each generated harmony color is shown as a **mini animated jellyfish** in that color
- Users can save individual harmony colors to any palette

---

### Random & Smart Color Generator

A dedicated card in the Palette Studio tab:

```
┌─────────────────────────────────────────────────────┐
│  ✨ Color Generator                                  │
│─────────────────────────────────────────────────────│
│  [ 🎲 Random Color ]                                 │
│     Generates a random color, displays result jelly  │
│                                                      │
│  [ 🧠 Smart Suggest ]                                │
│     Select a base color: [Palette dropdown]  [Pick]  │
│     Suggests colors that pair beautifully with       │
│     your chosen color, based on:                     │
│       ○ Complementary contrast                       │
│       ○ Analogous harmony                            │
│       ○ Neutral balance                              │
│     (Algorithm: HSL-based contrast + harmony rules)  │
│                                                      │
│  Result: [Jellyfish swatch grid]                     │
│  [ Save All ]  [ Save Selected ]                     │
└─────────────────────────────────────────────────────┘
```

**Random Color Generator:**
- Generates a random HSL color (can be toggled to bias toward pastels, vivids, or earth tones)
- Displays result as animated jellyfish with HEX + name
- One-click save to any palette

**Smart Suggest Algorithm:**
- User selects a base color (from saved palettes or by entering a HEX)
- App generates 4–6 suggested colors using:
  - HSL complementary rotation
  - Saturation and lightness balancing rules
  - Avoiding "muddy" mid-tone traps
- Results shown as a jellyfish swatch row
- Each suggestion has its own Save button

---

## Animations & Interactions

### Jellyfish Idle Animation
- **Bell pulse**: slow rhythmic contraction/expansion (CSS keyframe, ~3–5s cycle, eased)
- **Vertical drift**: gentle up-and-down float (sinusoidal, each jelly on a slightly different phase)
- **Lateral sway**: very subtle left-right rocking
- **Tentacle flow**: SVG path animation or CSS transform — tentacles trail and wave lazily

### Drag Interaction
- On mousedown / touchstart: jellyfish "wakes up" (slight scale-up, glow intensifies)
- Dragging: jellyfish follows cursor smoothly, tentacles lag behind with elastic interpolation
- On overlap: overlap region renders blended color in real time, both bells gain a halo effect at the intersection
- On release: jellyfish gently "settles" back into float animation at its new position

### Slider Interaction
- Moving a slider animates the corresponding jellyfish gliding toward or away from others
- Color result updates in real-time with a smooth color transition (CSS transition on the result jelly)

### Save Animation
- When color is saved: small burst of bubbles from the Save button, a tiny jellyfish icon flies into the Palette tab

### Tab Transition
- Smooth crossfade with a brief underwater ripple effect

### Hover States
- Jellyfish glow intensifies on hover
- UI cards lift slightly (translateY -2px) with shadow deepening
- Buttons shimmer with a light-sweep animation

---

## Data Model

```typescript
// Color
interface Color {
  hex: string;           // e.g. "#A040FF"
  name: string;          // e.g. "Amethyst Glow"
  rgb: { r: number; g: number; b: number };
  cmyk: { c: number; m: number; y: number; k: number };
}

// Palette
interface Palette {
  id: string;
  name: string;
  colors: Color[];
  createdAt: Date;
  updatedAt: Date;
}

// Mixer State
interface MixerState {
  mode: "RGB" | "CMYK";
  jellies: {
    id: string;
    baseColor: Color;
    x: number;
    y: number;
    intensity: number;  // 0–1, controlled by slider
  }[];
  result: Color | null;
}

// App State
interface AppState {
  activeTab: "mixer" | "palette";
  mixer: MixerState;
  palettes: Palette[];
  selectedPalette: string | null;
}
```

**Persistence:** All palette data stored in `localStorage` (JSON serialized). Export functions allow backup/import.

---

## Technical Stack

| Concern | Technology |
|---------|------------|
| Framework | React (Vite) or vanilla JS + CSS |
| Animations | CSS keyframes + Web Animations API, optionally GSAP |
| Canvas/SVG | SVG for jellyfish shapes, Canvas for particle effects |
| Color math | Custom utilities for RGB add, CMYK subtract, HSL harmony |
| State | React useState/useReducer or Zustand |
| Persistence | localStorage |
| Styling | CSS Modules or Tailwind + custom Frutiger Aero design tokens |
| Fonts | Nunito or Comfortaa (Google Fonts) |

### Color Math Notes

**RGB Additive Mixing (light):**
```
result.r = clamp(a.r + b.r, 0, 255)
result.g = clamp(a.g + b.g, 0, 255)
result.b = clamp(a.b + b.b, 0, 255)
```
Weighted by overlap intensity (0–1 sliders).

**CMYK Subtractive Mixing (ink):**
```
// Convert each to RGB, multiply channels (subtractive blend), convert back
result.r = (a.r / 255) * (b.r / 255) * 255
```
Or work natively in CMYK:
```
result.c = clamp(a.c + b.c, 0, 100)
// etc., then convert final CMYK → RGB for display
```

**Color Harmonies (HSL):**
```
complementary: hue + 180°
analogous: hue ± 30°
triadic: hue, hue + 120°, hue + 240°
split-complementary: hue, hue + 150°, hue + 210°
tetradic: hue, hue + 90°, hue + 180°, hue + 270°
```

---

## Accessibility

- All interactive elements keyboard-navigable (Tab, Enter, Space)
- Jellyfish drag also controllable via arrow keys when focused
- ARIA labels on all icon buttons
- Color results always display text (HEX, name) — never color alone
- High-contrast mode: darker backgrounds, stronger text contrast
- Reduced motion: disable all animations when `prefers-reduced-motion: reduce` is set; replace with instant state transitions
- All sliders have labeled values (e.g., `aria-label="Red Intensity: 128"`)

---

## Stretch Challenges

### Contrast Checker

A dedicated tool (accessible from the Palette Studio tab or as a standalone panel) that evaluates the accessibility of any two colors placed together.

```
┌──────────────────────────────────────────────────────────┐
│  ♿ Contrast Checker                                       │
│──────────────────────────────────────────────────────────│
│  Foreground: [color picker]  #1A1A2E                     │
│  Background: [color picker]  #E0F7FA                     │
│──────────────────────────────────────────────────────────│
│  Contrast Ratio:  12.3 : 1                               │
│                                                           │
│  Normal Text    AA ✅  AAA ✅                              │
│  Large Text     AA ✅  AAA ✅                              │
│  UI Components  AA ✅                                     │
│──────────────────────────────────────────────────────────│
│  [Preview: Sample text on background]                    │
└──────────────────────────────────────────────────────────┘
```

**WCAG thresholds:**

| Level | Normal Text | Large Text | UI / Graphics |
|-------|-------------|------------|---------------|
| AA    | ≥ 4.5 : 1  | ≥ 3.0 : 1 | ≥ 3.0 : 1    |
| AAA   | ≥ 7.0 : 1  | ≥ 4.5 : 1 | N/A           |

- Contrast ratio calculated using WCAG relative luminance formula
- Live preview renders sample body text and large heading text in the chosen foreground on the chosen background
- Both foreground and background can be loaded from any saved palette color
- Ratio and pass/fail badges update instantly on every color change

---

### Color Blindness Simulator

Available in the Palette Studio tab. Simulates how a saved palette (or any set of colors) appears to users with color vision deficiency.

```
┌──────────────────────────────────────────────────────────┐
│  👁 Color Blindness Simulator                             │
│──────────────────────────────────────────────────────────│
│  Palette: [dropdown]  "Ocean Dreams"                     │
│──────────────────────────────────────────────────────────│
│  Normal Vision     🪼 🪼 🪼 🪼 🪼                         │
│  Protanopia        🪼 🪼 🪼 🪼 🪼  (red-blind)            │
│  Deuteranopia      🪼 🪼 🪼 🪼 🪼  (green-blind)          │
│  Tritanopia        🪼 🪼 🪼 🪼 🪼  (blue-blind)           │
└──────────────────────────────────────────────────────────┘
```

**Simulation algorithm:** Apply the standard LMS-based color transformation matrices for each deficiency type to convert sRGB → simulated sRGB. Each row of swatches shows the palette colors as perceived under that condition.

**Deficiency types:**

| Type | Affected | Description |
|------|----------|-------------|
| Protanopia | L-cones (red) | Reds appear dark/brown; reds and greens confused |
| Deuteranopia | M-cones (green) | Greens appear olive/yellow; reds and greens confused |
| Tritanopia | S-cones (blue) | Blues appear green; yellows and purples confused |

---

### Accessible Palette Mode

An option in the **Random & Smart Color Generator** that constrains generated colors so they maintain minimum contrast ratios against each other or against a specified background.

```
┌──────────────────────────────────────────────────────────┐
│  ♿ Accessible Palette Mode                  [ ON / OFF ] │
│──────────────────────────────────────────────────────────│
│  Minimum contrast target:                                 │
│    ○ AA (4.5:1)   ● AAA (7.0:1)                         │
│  Check contrast against:                                  │
│    ○ Each other   ● Background  [color picker] #FFFFFF   │
└──────────────────────────────────────────────────────────┘
```

- When **ON**, the generator rejects and resamples any candidate color that fails the selected contrast threshold
- Works with both Random Color and Smart Suggest generators
- Each generated swatch displays its contrast ratio badge
- Helps designers build inclusive, WCAG-compliant color palettes

---

## Out of Scope (v1.0)

- User accounts / cloud sync (palettes are local only)
- LAB / HSL color models (future)
- Mobile-optimized drag (touch events are supported but not the primary design target)
- AI-generated palette names (future — for now uses curated lookup table)

---

*Spec authored for Moon Jelly Color Mixer v1.0 — go build something beautiful 🪼*
