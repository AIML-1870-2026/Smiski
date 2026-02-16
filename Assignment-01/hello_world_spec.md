# Hello World Spec

## Overview
A whimsical, animated "Hello, World!" webpage featuring a winter night scene with aurora effects, falling snowflakes, silhouetted pine trees, and a cozy cottage.

## Visual Design

### Color Palette
- **Primary Colors:**
  - Pink: `#FFB3D9`, `#FFE5F0`, `#FF69B4`
  - Cream: `#FFF9F0`
  - Brown: `#8B4513`
  - Green: `#90EE90`
  - Blue Sky: `#B3E5FC`

- **Background:**
  - Deep blue gradient base: `#1a1a2e` → `#16213e` → `#0f3460`
  - Starry night effect with 40+ radial gradient stars scattered across viewport
  - Aurora borealis overlays with purple and pink gradients

### Typography
- **Heading Font:** 'Chewy' (cursive, playful)
- **Body Font:** 'Nunito' (sans-serif, clean)
- **Main Heading:** "Hello, World!" at 3.5rem (2.5rem mobile)
- **Subtitle:** "I'm ready to vibe code." at 2rem (1.5rem mobile)

## Animations

### Aurora Effects
Two overlapping aurora gradients animate across the screen:
- **Aurora 1:** Horizontal wave (15s cycle, purple/pink)
- **Aurora 2:** Vertical wave (20s cycle, cyan/purple/pink)
- Effects use `opacity` and `transform` with `ease-in-out` timing

### Snowflakes
- 20 individual snowflakes with unique timing
- Fall from top with rotation animation
- Durations: 9-13 seconds
- Varying delays (0-7 seconds)
- Fade out at 65% of animation cycle
- Font sizes: 0.8rem - 1.2rem

### Interactive Elements
- **Bunny Emoji:** Gentle bounce animation (5rem size, 2s cycle)
- **Sparkles:** Four twinkling stars around card with fade animations
- **Floating Flowers/Hearts:** Scattered decorative emojis around content

## Scene Elements

### Background Scene
- **Moon:** Fixed position (top-right), 100px diameter with shadow effects and crater details
- **Stars:** 40+ twinkling points created with radial gradients
- **Ground Layer:** Translucent white gradient at bottom

### Silhouettes (Bottom Layer)
All elements positioned as dark silhouettes against the night sky:

- **Hill:** Rolling contour across bottom
- **Pine Trees:** 8 trees with varying heights (140-220px)
  - Triangular branches created with CSS borders
  - Small trunk at base
  - Positions: scattered across width (left: 10-95%)
- **Cottage:**
  - Two-story structure (right side)
  - Sloped roof with chimney
  - Animated smoke puffs rising from chimney
  - Glowing windows (yellow light: `#F5DEB3`)
  - Small door detail

### Content Card
Centered card with pink gradient background and rounded corners:
- Background: `#FFB3D9` to `#FFE5F0` gradient
- Border radius: 30px
- Shadow: Multi-layer drop shadows for depth
- Padding: 4rem (3rem mobile)
- Pop-in animation on load (0.6s delay)

## Responsive Behavior

### Desktop (> 768px)
- Full-size elements and typography
- Moon: 100px diameter at top-right
- Card: 4rem padding

### Mobile (≤ 768px)
- Reduced font sizes (heading: 2.5rem, subtitle: 1.5rem)
- Smaller decorative elements (bunny: 4rem, sparkles: 1rem)
- Card: 3rem padding with 1rem margin
- Moon: 70px diameter, repositioned closer to corner

## Technical Implementation

### HTML Structure
- Minimal markup with semantic sections
- Snowflakes: 20 div elements with `.snowflake` class
- Scene layers: `.silhouettes`, `.ground`, `.moon`
- Content: `.container` > `.card`

### CSS Architecture
- CSS custom properties for theme colors
- Keyframe animations for all motion
- Pseudo-elements (`:before`, `:after`) for aurora overlays
- `position: fixed` for scene elements
- `z-index` layering: aurora (1) → moon/stars (2) → snowflakes (5) → content (10)

### Performance
- Hardware-accelerated transforms
- Efficient animation loops
- No JavaScript required
- Single HTML file (all-in-one)

## Browser Compatibility
- Modern browsers supporting CSS Grid, transforms, and animations
- Google Fonts for web typography
- Graceful degradation for older browsers

## File Structure
Single self-contained HTML file with:
- Inline CSS in `<style>` block
- Google Fonts import
- No external dependencies beyond fonts
