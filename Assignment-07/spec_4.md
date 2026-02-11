# Julia Set Explorer — spec.md

## Overview
An interactive, visually stunning Julia Set fractal explorer built as a single-page HTML/CSS/JS web application. The explorer lets users manipulate fractal parameters in real time, zoom into infinite detail, and discover the connection between the Mandelbrot set and Julia sets.

## Aesthetic Direction
**Theme:** Dark cosmic / mission control — deep space blacks (#0a0a0f) with vibrant neon fractal renders. The UI should feel like piloting a spacecraft through mathematical infinity.

**Typography:** "JetBrains Mono" for data/coordinates, "Space Grotesk" for headings, system sans-serif fallback.

**Color Palette:**
- Background: #0a0a0f (near-black)
- Panel backgrounds: rgba(255,255,255,0.04) with subtle borders
- Accent: #00e5ff (cyan glow)
- Secondary accent: #ff6ec7 (hot pink)
- Text: #e0e0e0 (soft white)

## Features

### Core (Required)
1. **Real-time Julia Set rendering** on an HTML5 Canvas using the escape-time algorithm
2. **Interactive c-parameter control** — click on a mini Mandelbrot set to pick `c`, OR use sliders for Real and Imaginary parts
3. **Multiple color schemes** — at least 5 selectable palettes (Inferno, Ocean, Neon, Grayscale, Psychedelic)
4. **Zoom & Pan** — mouse wheel to zoom, click-drag to pan, pinch-zoom on mobile
5. **Preset buttons** for famous Julia sets: Dendrite, Spiral, Douady Rabbit, San Marco, Dragons
6. **Max iteration slider** (50–1000) to control detail level
7. **Save/Download** button to export the current view as a PNG

### Stretch Goals
8. **Mandelbrot–Julia split view**: side-by-side panels; clicking on the Mandelbrot set updates the Julia set in real time
9. **Parameter morphing animation**: animate `c` along a circular or custom path through the complex plane
10. **Burning Ship fractal** toggle mode
11. **Coordinate display**: show current mouse position in the complex plane
12. **Reset view** button to return to default zoom/position

## UI Layout

```
┌──────────────────────────────────────────────────────┐
│  Header: "Julia Set Explorer" + subtitle             │
├────────────────────────────┬─────────────────────────┤
│                            │  Controls Panel         │
│                            │  ─────────────────────  │
│                            │  c = Re + Im i          │
│    Main Canvas             │  [slider Re] [-2 to 2]  │
│    (Julia Set render)      │  [slider Im] [-2 to 2]  │
│                            │  ─────────────────────  │
│                            │  Presets: [btn] [btn]..  │
│                            │  ─────────────────────  │
│                            │  Color Scheme: [select]  │
│                            │  Max Iterations: [sldr]  │
│                            │  ─────────────────────  │
│                            │  Mini Mandelbrot canvas  │
│                            │  (click to pick c)       │
│                            │  ─────────────────────  │
│                            │  [Animate] [Save] [Reset]│
│                            │  ─────────────────────  │
│                            │  Mouse: z = x + yi       │
├────────────────────────────┴─────────────────────────┤
│  Footer: coordinates, zoom level                      │
└──────────────────────────────────────────────────────┘
```

On mobile (<768px), the controls panel moves below the canvas in a collapsible drawer.

## Interaction Behaviors
- **Sliders** update the fractal in real time (debounced to ~30fps)
- **Mouse wheel** over main canvas zooms toward cursor position
- **Click-drag** on main canvas pans the view
- **Click** on mini Mandelbrot sets `c` and re-renders Julia set
- **Preset buttons** animate a smooth transition to the preset's `c` value
- **Animate button** toggles parameter morphing along a circle in the complex plane
- **Save button** triggers a PNG download of the main canvas
- **Reset button** restores default zoom, pan, and `c` value

## Technical Requirements
- Single HTML file (inline CSS + JS) — no build tools, no dependencies
- Web Workers for fractal computation to keep UI responsive
- Canvas API for rendering
- Runs smoothly in Safari, Chrome, Firefox
- Responsive layout for desktop and tablet
