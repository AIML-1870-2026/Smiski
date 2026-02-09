# Turing Patterns Explorer - Project Specification

## Project Overview
Create an interactive web application for exploring reaction-diffusion systems (Turing patterns). The app should allow users to visualize, customize, and interact with beautiful mathematical patterns that emerge from chemical reaction simulations.

## Page Layout

### Overall Structure
- **Long-scroll layout**: The canvas appears first (hero section), with controls below
- **Dynamic background**: Background color theme matches the selected color scheme
- **Responsive design**: Should work well on desktop and tablet devices

### Hero Section (Above the fold)
1. **Title**: "Turing Patterns Explorer" (prominent, centered)
2. **Subtitle/Description**: A brief 2-3 sentence explanation:
   - "Explore the mesmerizing world of reaction-diffusion systems. These mathematical patterns emerge from simple chemical reactions, creating organic, nature-like forms."
   - "Click on the canvas to add spawn points and watch patterns emerge. Scroll down to customize your simulation."
3. **Canvas**: Large, centered simulation canvas (primary visual focus)
   - Should be interactive (clickable to add spawn points)
   - Dimensions: Recommend 800x600px or similar prominent size

### Controls Section (Below the fold, revealed on scroll)
All control panels should be well-organized, clearly labeled sections.

---

## Core Features

### 1. Simulation Models
**Model Selection Dropdown**
- Gray-Scott (default)
- Brusselator
- Schnakenberg

Each model uses sub-stepping with model-specific time steps (dt) and diffusion ratios to ensure numerical stability and visible Turing pattern formation. Slider values are mapped internally to each model's native parameters. Switching models resets sliders to that model's defaults and re-initializes the grid to the correct steady state.

### 2. Color Schemes
**Color Scheme Selector** (dropdown or button group)
- **Fire**: Warm colors (deep red → orange → yellow → white)
- **Ice**: Cool colors (dark blue → cyan → light blue → white)
- **Ocean**: Blues, greens, and subtle yellows (deep blue → teal → aqua → pale yellow)
- **Purple**: User's favorite! (dark purple → magenta → lavender → white)
- **Rainbow/Heat Map**: Full spectrum (violet → blue → cyan → green → yellow → orange → red)

**Important**: The selected color scheme should also influence the page background (using a subtle, darker version of the scheme's base color).

### 3. Simulation Controls
**Control Panel Box**
- **Pause/Resume Button**: Toggle simulation on/off (button text changes based on state)
- **Reset Button**: Restart simulation with current parameters (keeps spawn points)
- **Clear All Button**: Completely clear the canvas and remove all spawn points
- **Save Button**: Opens a modal/dialog with two options:
  - "Save as Video" - Records and downloads the simulation as MP4/WebM
  - "Save as Image" - Downloads current frame as PNG

### 4. Parameter Controls
**Parameters Box**
Sliders with real-time value display:
- **Feed Rate**: Range [0.01 - 0.10], step 0.001, default varies by model
- **Kill Rate**: Range [0.03 - 0.07], step 0.001, default varies by model
- **Simulation Speed**: Range [0.1x - 5.0x], step 0.1, default 1.0x

Note: Parameters should be adjustable before and during simulation.

### 5. Animation Modes
**Animation Selector** (dropdown)
Applied once per frame before simulation stepping:
- **Standard**: Normal reaction-diffusion evolution (no extra effects)
- **Pulse**: Expanding ring perturbations emanate from spawn points every 8 frames; full re-seeding with radius 8 every 25 frames. Ring radius cycles over 80 frames. Uses model-appropriate perturbation amplitudes (Gray-Scott: v += 0.2, others: ±1.5 random).
- **Spiral**: Global rotational advection applied to every cell each frame. Reads from a snapshot of the grid, then blends each cell toward its rotational neighbor (perpendicular to the center vector) at strength 0.07. Creates visible swirling/vortex patterns.
- **Chaotic**: Injects random noise into 1500 cells per frame. Amplitude is model-aware (Gray-Scott: ±0.15, Brusselator/Schnakenberg: ±1.5) to account for different value scales.
- **Freeze-Frame**: Stops auto-advancing; reveals a "Step Forward" button for manual single-step control.

### 6. Interactive Canvas
**Click-to-Add Spawn Points**
- Users can click anywhere on the canvas to add new reaction centers
- Visual indicator (small circle/dot) shows spawn point locations
- Patterns should grow/spread from these points
- Each spawn point should trigger the reaction-diffusion from that location

### 7. Kaleidoscope Mode
**Toggle Button**: "Kaleidoscope Mode ON/OFF"

When enabled:
- Apply radial/mirror symmetry to the pattern (4-fold, 6-fold, or 8-fold symmetry)
- Pattern in one "wedge" is reflected across multiple axes
- Creates mandala-like, highly symmetrical visuals
- Should maintain reactivity to spawn points

---

## Technical Requirements

### Implementation Stack
- **HTML/CSS/JavaScript** (vanilla or with lightweight framework)
- **Canvas API** or **WebGL** for rendering (WebGL recommended for performance)
- **MediaRecorder API** for video saving functionality

### Performance Considerations
- Simulation should run at 30-60 FPS on modern hardware
- Gray-Scott uses dt=1.0 with 1 sub-step (lightweight). Brusselator/Schnakenberg use dt=0.02 with 10 sub-steps per step (required for numerical stability with large diffusion ratios). At speed 1.0x with 5 steps/frame, this means ~50 grid sweeps/frame for those models (~1.5M cell updates/frame).
- Typed arrays (Float32Array) for simulation grids and color LUT (Uint8Array) for fast rendering
- Offscreen canvas rendering at grid resolution (200x150), then scaled up to display canvas (800x600) to minimize per-pixel work

### Algorithm Notes

**Grid**: 200x150 simulation grid rendered to 800x600 canvas with bilinear upscaling.
**Laplacian**: Weighted 3x3 kernel (0.05 corners, 0.2 edges, -1.0 center) with toroidal (wrapping) boundary conditions.
**Rendering**: Adaptive min/max normalization of `v` concentration mapped through a 256-entry color LUT. Works automatically across all models regardless of value range.
**Stability**: Value clamping [-10, 50] prevents numerical blowup at extreme parameter settings.

**Gray-Scott Model** (default):
```
∂u/∂t = Du∇²u - uv² + F(1-u)
∂v/∂t = Dv∇²v + uv² - (F+k)v
```
- Du = 0.21, Dv = 0.105, dt = 1.0, 1 sub-step per step
- F = feed rate (slider direct), k = kill rate (slider direct)
- Default: F = 0.055, k = 0.062
- Init: u = 1.0, v = 0.0 everywhere; spawn points set u = 0.5, v = 0.25

**Brusselator Model**:
```
∂u/∂t = Du∇²u + A - (B+1)u + u²v
∂v/∂t = Dv∇²v + Bu - u²v
```
- Du = 1.0, Dv = 16.0 (ratio 16 required for Turing instability), dt = 0.02, 10 sub-steps per step
- A = 1 + feedRate × 30, B = killRate × 100
- Default: feed = 0.040 → A = 2.2, kill = 0.052 → B = 5.2
- Init: u = A, v = B/A (steady state) + small random noise (±0.1)
- Turing condition: B must be between B_crit and 1+A² for pattern formation

**Schnakenberg Model**:
```
∂u/∂t = Du∇²u + a - u + u²v
∂v/∂t = Dv∇²v + b - u²v
```
- Du = 1.0, Dv = 20.0 (ratio 20), dt = 0.02, 10 sub-steps per step
- a = feedRate × 5, b = killRate × 20
- Default: feed = 0.040 → a = 0.20, kill = 0.060 → b = 1.20
- Init: u = a+b, v = b/(a+b)² (steady state) + small random noise (±0.1)

### Visual Polish
- Smooth color gradients (use interpolation)
- Anti-aliasing if possible
- Subtle animations for UI controls (hover effects, transitions)
- Loading indicator when simulation initializes
- Tooltips on sliders showing current values

---

## User Experience Flow

1. **Landing**: User sees title, description, and canvas with a default pattern beginning to form
2. **Explore**: User clicks canvas to add spawn points and watches patterns grow
3. **Scroll**: User scrolls down to discover customization options
4. **Customize**: User adjusts model, colors, parameters, and animation mode
5. **Reset/Tweak**: User resets and experiments with different settings
6. **Save**: User captures their favorite creation as video or image

---

## File Structure
Single-file implementation (per project coding standards):
```
index.html          # Complete application: HTML structure, embedded CSS, and all JavaScript
```
All styles (CSS custom properties for theming, glassmorphism panels, responsive layout) and scripts (simulation engine, renderer, UI controls, save functionality) are contained in the single HTML file. Google Fonts loaded externally (Poppins, Roboto, Roboto Mono).

---

## Additional Notes

- **Accessibility**: Ensure all controls have proper labels and keyboard navigation
- **Mobile Consideration**: While primarily desktop-focused, basic mobile touch support would be nice
- **Browser Compatibility**: Target modern browsers (Chrome, Firefox, Safari, Edge)
- **Default State**: Starts with Gray-Scott model, Fire color scheme, speed 1.0x, 7 spawn points near center. Space bar toggles pause/resume.

---

## Visual Design Notes

### Color Scheme Palettes (Suggested)
**Fire**: `['#0a0000', '#4a0000', '#8b0000', '#ff4500', '#ff8c00', '#ffd700', '#ffffe0']`
**Ice**: `['#000033', '#001a4d', '#0047ab', '#4682b4', '#87ceeb', '#b0e0e6', '#f0ffff']`
**Ocean**: `['#001f3f', '#003d5c', '#006994', '#0088a8', '#1ca3c4', '#5dbcd2', '#ffffcc']`
**Purple**: `['#1a001a', '#330033', '#4b0082', '#8b008b', '#ba55d3', '#da70d6', '#f8f8ff']`
**Rainbow**: `['#4b0082', '#0000ff', '#00ffff', '#00ff00', '#ffff00', '#ff7f00', '#ff0000']`

### Typography
- Title: Large, modern sans-serif (consider Poppins, Inter, or Montserrat)
- Body text: Readable, clean (Roboto, Open Sans, or system font)
- Monospace for parameter values (Consolas, Monaco, or 'Courier New')

### Layout Spacing
- Hero section: Full viewport height
- Control sections: Generous padding, clear visual separation
- Canvas: Centered with subtle shadow/border

---

## Success Criteria
- ✅ Smooth, real-time reaction-diffusion simulation
- ✅ All five color schemes working and affecting background
- ✅ Click-to-add spawn points functional
- ✅ All parameters adjustable with immediate visual feedback
- ✅ Video and image saving functional
- ✅ Kaleidoscope mode creates beautiful symmetrical patterns
- ✅ UI is intuitive and visually appealing
- ✅ Performance remains smooth even with multiple spawn points

---

## Optional Enhancements (If Time Permits)
- Preset patterns gallery (famous Turing pattern configurations)
- Share button (generate URL with current settings)
- Fullscreen mode for canvas
- Multiple kaleidoscope symmetry options (4-fold, 6-fold, 8-fold)
- "Random" button that generates random interesting parameters
- Pattern history/undo functionality

---

**End of Specification**

Good luck with implementation! This should be a stunning visual experience. 🎨✨
